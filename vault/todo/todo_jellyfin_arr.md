# Jellyfin + Arr Stack

## Decision: Single Privileged LXC with Docker

Docker-in-LXC research (Jul 2026):

- **Unprivileged** LXC + Docker is broken on modern kernels (runc 1.2+): sysctl writes and AppArmor probes can't work through the user namespace — a hard ceiling, not configurable
- **Privileged** LXC + Docker works reliably with `nesting=1,keyctl=1,fuse=1` and `lxc.apparmor.profile: unconfined`
- VM would be more isolated but costs ~512MB+ RAM overhead vs \~100MB for LXC; with 16GB total and ~9GB already used, LXC wins on density
- This is a homelab behind LAN/Tailscale — the security trade-off (privileged LXC = container root = host root) is acceptable for internal services where you control all images

## Specs

- Type: Privileged LXC
- IP: 192.168.1.25
- Cores: 4
- RAM: 4096 MB
- Swap: 1024 MB
- Disk: 32 GB
- OS: Ubuntu 24.04 LTS
- Features: nesting=1, keyctl=1, fuse=1
- AppArmor: unconfined

## LXC Config (`/etc/pve/lxc/<ID>.conf`)

```
features: nesting=1,keyctl=1,fuse=1
lxc.apparmor.profile: unconfined
lxc.cap.drop:
```

## Storage Mounts

```
pct set <ID> -mp0 /mnt/storage/media,mp=/mnt/media
pct set <ID> -mp1 /mnt/storage/downloads,mp=/mnt/downloads
```

Fix ownership for unprivileged bind mount: `chown -R 101000:101000 /mnt/storage/media /mnt/storage/downloads`

## Stack Overview

### Jellyfin

- Media server — streams movies, TV, and music to any device
- Port: 8096
- Config: `/opt/jellyfin`
- Media: `/mnt/media`
- GPU transcoding can be added later via `/dev/dri` passthrough in the LXC config

### VPN Gateway

- **Gluetun** — Docker container that acts as a WireGuard/OpenVPN gateway. qBittorrent uses `network_mode: "service:gluetun"` so all its traffic routes through the VPN tunnel. Only qBittorrent is routed through Gluetun — Sonarr/Radarr/Jellyfin stay on the normal network so they can reach indexers and local clients.
- Recommendation: **ProtonVPN Plus** (~$5.99/mo) — audited no-log, Swiss jurisdiction, works seamlessly with Gluetun, supports port forwarding (essential for seeding on private trackers)
- Alternative: **AirVPN** (~€3.50/mo) — up to 5 **static** forwarded ports that persist across reboots (no sync tool needed), accepts BTC/cash
- Avoid for seeding: Mullvad (removed port forwarding 2023), NordVPN (never had it)

### Download Client

- **qBittorrent** — torrent client with built-in search
- Port: 8080 (web UI on host), 6881 (torrent traffic)
- Downloads to: `/mnt/downloads/incomplete` → `/mnt/downloads/complete`
- Runs inside Gluetun's network namespace — no direct internet access, fully isolated behind VPN

### Indexer Manager

- **Prowlarr** — single proxy for all torrent/usenet indexers. You add indexers here once and they sync automatically to Sonarr, Radarr, Lidarr, and Readarr
- Port: 9696

### Media Managers

- **Sonarr** — TV show management. Monitors shows you want, searches for new episodes via Prowlarr, sends them to qBittorrent, then renames and organizes them into `/mnt/media/tv`
- Port: 8989
- **Radarr** — Movie management. Same workflow as Sonarr for movies into `/mnt/media/movies`
- Port: 7878
- **Lidarr** — Music management. Monitors artists/albums, downloads and organizes into `/mnt/media/music`
- Port: 8686
- **Readarr** — Book and audiobook management (optional). Same pattern for ebooks/audiobooks into `/mnt/media/books`
- Port: 8787

### Subtitle Management (optional)

- **Bazarr** — automatically downloads subtitles for your Sonarr/Radarr media libraries
- Port: 6767

### Request Management (optional)

- **Jellyseerr** — clean web UI for users to browse and request movies/TV shows. Integrates with Sonarr/Radarr for downloads and Jellyfin for media visibility
- Port: 5055

### Utility (optional)

- **Watchtower** — automatically updates running Docker images when new versions are released
- **FlareSolverr** — proxy that solves Cloudflare challenges for indexers that require it
- Port: 8191

## Data Flow

```
  Internet
      |
      ▼
   Gluetun ─── ProtonVPN tunnel ─── qBittorrent
      |
      ▼
/mnt/downloads/complete
      |
      ▼  (Sonarr/Radarr rename + move)
      |
/mnt/media/{tv,movies,music}
      |
      ├── Jellyfin ←── Users
      └── Bazarr (subtitles)

Indexers via Prowlarr ─── Sonarr/Radarr/Lidarr (normal network)
                                  ↕
                          Jellyseerr ←── user requests
```

## docker-compose.yml (structure)

All services in one compose file under a shared Docker network. Config stored in `/opt/docker/`.

VPN network:

- **Gluetun** — connects to ProtonVPN, exposes qBittorrent's web UI port
- **qBittorrent** — `network_mode: "service:gluetun"`, no own IP
- **qsticky** or **qb-port-sync** — sidecar that reads ProtonVPN's forwarded port from Gluetun's API and updates qBittorrent's listen port automatically

ProtonVPN setup requires a WireGuard private key from your ProtonVPN account dashboard. Add to `.env`:

```
WIREGUARD_PRIVATE_KEY=your_key_here
SERVER_COUNTRIES=Netherlands
GLUETUN_API_KEY=<openssl rand -hex 16>
QBITTORRENT_USER=admin
QBITTORRENT_PASS=your_password
```

```yaml
services:
  gluetun:     # no host port (see below)
  qbittorrent: # port 8080 (exposed via gluetun)
  qsticky:     # no port (auto-syncs forwarded port)
  prowlarr:    # port 9696
  sonarr:      # port 8989
  radarr:      # port 7878
  lidarr:      # port 8686
  readarr:     # port 8787 (optional)
  jellyfin:    # port 8096
  bazarr:      # port 6767 (optional)
  jellyseerr:  # port 5055 (optional)
  flaresolverr:# port 8191 (optional)
  watchtower:  # no port (auto-updater)
```

Services that route through the VPN (`network_mode: "service:gluetun"`) have their ports declared on the Gluetun service instead of directly:

```yaml
gluetun:
  ports:
    - 8080:8080/tcp # qBittorrent web UI

qbittorrent:
  network_mode: "service:gluetun"
  depends_on:
    gluetun:
      condition: service_healthy

qsticky:
  network_mode: "service:gluetun"
  depends_on:
    gluetun:
      condition: service_healthy
    qbittorrent:
      condition: service_healthy
```

All \*arr services and Jellyfin stay on the default bridge network (no VPN), so they remain accessible on LAN and can reach indexers directly.

## Glance Dashboard

Add widgets after setup:

- Jellyfin: `http://192.168.1.25:8096`
- Prowlarr: `http://192.168.1.25:9696`
- qBittorrent: `http://192.168.1.25:8080`
