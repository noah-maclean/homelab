---
title: Jellyfin + Arr Stack
created: 2026-06-16
type: todo
author: codex, hermes
tags:
  - todo
  - jellyfin
  - media
  - arr
---
# Jellyfin + Arr Stack

## Decision: Two LXCs — Jellyfin native + Arr stack in Docker

Research and decision (Aug 2026):

- **Docker-in-LXC vs Docker-in-VM** — both work; Proxmox's own docs recommend a VM for maximum isolation, but LXC is the mainstream homelab choice: ~100 MB RAM overhead vs ~1–2 GB for a VM, instant boot, and Intel iGPU (QSV) can be shared across multiple LXCs. With 24 GB total RAM on the host, LXC wins on density. A VM stays the fallback if live migration or stronger isolation is ever needed.
- **Unprivileged LXC + Docker is fine on PVE 9** — the old runc 1.2+ sysctl breakage (`net.ipv4.ip_unprivileged_port_start` permission denied) was a PVE 8-era problem. PVE 9 (Debian 13 kernel, LXC 6.0 profiles, newer runc/containerd) fixed it, so there is no need for a privileged LXC (where container root = host root). The community-scripts Docker installer creates an unprivileged LXC by default.
- **Split vs combined** — the \*arr suite belongs in ONE Docker compose: Sonarr/Radarr/Prowlarr/qBittorrent talk constantly, share UIDs on media folders, and one compose file pins versions. Jellyfin gets its own **native** LXC (no Docker): transcoding can't starve the download stack, media serving stays up during arr-stack maintenance, and it has a separate backup boundary.

## Specs

### LXC A — Jellyfin (community-scripts installer)

- Type: Unprivileged LXC, native Jellyfin install (no Docker)
- Install: `bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/jellyfin.sh)"`
- Defaults: 2 cores / 2048 MB RAM / 8 GB disk (adjust at prompt)
- OS: Debian 13 (Trixie)
- Port: 8096
- GPU transcoding can be added later via `/dev/dri` passthrough (Intel QSV)

### LXC B — Arr stack (community-scripts Docker installer)

- Type: Unprivileged LXC with Docker Engine + Compose
- Install: `bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/docker.sh)"`
- Defaults: 2 cores / 2048 MB RAM / 4 GB disk — bump to **4 cores / 4096 MB / 32 GB** at the prompt
- OS: Debian 13 (Trixie)
- Features: `nesting=1, keyctl=1, fuse=1` (set by the script)
- Config: `/opt/docker/` with one compose file

## Storage Mounts

Run on the Proxmox host shell (adjust IDs):

```shell
pct set <JELLYFIN_ID> -mp0 /mnt/storage/media,mp=/mnt/media,ro=1
pct set <ARR_ID> -mp0 /mnt/storage/media,mp=/mnt/media
pct set <ARR_ID> -mp1 /mnt/storage/downloads,mp=/mnt/downloads
```

Jellyfin gets media **read-only**; the arr stack owns downloads and writes into media.

Fix ownership for unprivileged bind mounts (host side, real root — never container root):

```shell
chown -R 101000:101000 /mnt/storage/media /mnt/storage/downloads
chmod -R 775 /mnt/storage/media /mnt/storage/downloads
```

`101000` maps to container UID 1000 (default subuid base 100000 + 1000), matching `PUID=1000`/`PGID=1000` for the Docker containers and keeping media world-readable for the Jellyfin service user.

## Stack Overview

### Jellyfin (LXC A)

- Media server — streams movies, TV, and music to any device
- Config: `/etc/jellyfin` (script default)
- Media: `/mnt/media` (read-only bind mount)

### VPN Gateway (LXC B)

- **Gluetun** — Docker container that acts as a WireGuard/OpenVPN gateway. qBittorrent uses `network_mode: "service:gluetun"` so all its traffic routes through the VPN tunnel. Only qBittorrent is routed through Gluetun — Sonarr/Radarr stay on the normal network so they can reach indexers and local clients.
- **ProtonVPN Plus confirmed (Aug 2026)** — supports port forwarding (Plus and above), works seamlessly with Gluetun's built-in NAT-PMP support.
- **No sidecar needed** — modern Gluetun handles ProtonVPN port forwarding natively. In the ProtonVPN dashboard, create a WireGuard config with **NAT-PMP (Port Forwarding) ON** on a P2P server, then copy the `PrivateKey` into `.env`. Gluetun writes the forwarded port and pushes it into qBittorrent automatically via `VPN_PORT_FORWARDING_UP_COMMAND`.
- Avoid for seeding: Mullvad (removed port forwarding 2023), NordVPN (never had it)

### Download Client (LXC B)

- **qBittorrent** — torrent client with built-in search
- Port: 8080 (web UI, exposed via gluetun), 6881 (torrent traffic)
- Downloads to: `/mnt/downloads/incomplete` → `/mnt/downloads/complete`
- Runs inside Gluetun's network namespace — no direct internet access, fully isolated behind VPN

### Indexer Manager (LXC B)

- **Prowlarr** — single proxy for all torrent/usenet indexers. You add indexers here once and they sync automatically to Sonarr, Radarr, Lidarr, and Readarr
- Port: 9696

### Media Managers (LXC B)

- **Sonarr** — TV show management. Monitors shows you want, searches for new episodes via Prowlarr, sends them to qBittorrent, then renames and organizes them into `/mnt/media/tv`
- Port: 8989
- **Radarr** — Movie management. Same workflow as Sonarr for movies into `/mnt/media/movies`
- Port: 7878
- **Lidarr** — Music management. Monitors artists/albums, downloads and organizes into `/mnt/media/music`
- Port: 8686
- **Readarr** — Book and audiobook management (optional). Same pattern for ebooks/audiobooks into `/mnt/media/books`
- Port: 8787

### Subtitle Management (optional, LXC B)

- **Bazarr** — automatically downloads subtitles for your Sonarr/Radarr media libraries
- Port: 6767

### Request Management (optional, LXC B)

- **Jellyseerr** — clean web UI for users to browse and request movies/TV shows. Integrates with Sonarr/Radarr for downloads and Jellyfin for media visibility
- Port: 5055

### Utility (optional, LXC B)

- **Watchtower** — automatically updates running Docker images when new versions are released
- **FlareSolverr** — proxy that solves Cloudflare challenges for indexers that require it
- Port: 8191

## Data Flow

```text
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
/mnt/storage/media/{tv,movies,music}
      |
      ├── Jellyfin (LXC A, read-only) ←── Users
      └── Bazarr (subtitles)

Indexers via Prowlarr ─── Sonarr/Radarr/Lidarr (normal network)
                                  ↕
                          Jellyseerr ←── user requests
```

## docker-compose.yml (structure, LXC B)

All services in one compose file under a shared Docker network. Config stored in `/opt/docker/`.

VPN network:

- **Gluetun** — connects to ProtonVPN (WireGuard + NAT-PMP), exposes qBittorrent's web UI port
- **qBittorrent** — `network_mode: "service:gluetun"`, no own IP
- Gluetun auto-syncs ProtonVPN's forwarded port into qBittorrent via `VPN_PORT_FORWARDING_UP_COMMAND` (no sidecar container)

ProtonVPN setup requires a WireGuard private key from the ProtonVPN dashboard (config created with NAT-PMP enabled). Add to `.env`:

```env
VPN_SERVICE_PROVIDER=protonvpn
VPN_TYPE=wireguard
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
  prowlarr:    # port 9696
  sonarr:      # port 8989
  radarr:      # port 7878
  lidarr:      # port 8686
  readarr:     # port 8787 (optional)
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
```

All \*arr services stay on the default bridge network (no VPN), so they remain accessible on LAN and can reach indexers directly.

## [[glance|Glance Dashboard]]

Add widgets after setup (IPs TBD once the LXCs are created — note `.25` is [[immich|Immich]], not this stack):

- Jellyfin: `http://<jellyfin-lxc-ip>:8096`
- Prowlarr: `http://<arr-lxc-ip>:9696`
- qBittorrent: `http://<arr-lxc-ip>:8080`

## Related

- [[glance]]
- [[immich]]
- [[todo_proxmox]]
- [[todo_reverse_proxy]]
- [[todo_containers]]
- [[goals]]
