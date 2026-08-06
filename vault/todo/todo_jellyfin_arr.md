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

## Decision: One VM running the complete Docker stack

Research and decision (Aug 2026):

- **Docker-in-LXC vs Docker-in-VM** — both can work, but Proxmox recommends a VM for application containers such as Docker. A VM avoids LXC-specific AppArmor, cgroup, UID-mapping, and nested-namespace issues and is the simplest long-term setup. The extra memory overhead is acceptable for this host.
- **Unprivileged LXC + Docker is not categorically broken**, but it needs more careful configuration and can encounter compatibility problems after kernel, Proxmox, Docker, or `runc` upgrades. Rootless Docker also has limitations that make the Gluetun/qBittorrent VPN arrangement less attractive.
- **One Compose project** — Jellyfin and the \*arr services share one Docker Compose project. They use consistent paths and permissions, and the whole stack can be backed up or moved together.
- **LXC remains a fallback** — if memory becomes tight, an unprivileged LXC can still host the stack, but it is no longer the recommended first implementation.

## Specs

### Docker VM

- Type: Debian or Ubuntu Server VM with Docker Engine + Compose
- Resources: **4 cores / 4096–6144 MB RAM / 32 GB system disk**
- GPU transcoding can be added later by passing `/dev/dri` through to the VM (Intel QSV)
- Config: `/opt/docker/` with one compose file

## Storage Mounts

Unlike an LXC, a VM cannot directly use `pct set -mp0` bind mounts. Mount the storage inside the guest instead. The simplest choice is to export `/mnt/storage` from a storage server/NAS using NFS and mount it once in the VM, or attach the storage disk directly to the VM. Keeping media and downloads under one mount preserves hardlinks and atomic moves.

Example NFS mounts inside the VM:

```shell
sudo mkdir -p /mnt/data
sudo mount -t nfs <STORAGE_HOST>:/mnt/storage /mnt/data
```

Add the mounts to `/etc/fstab` after testing. Jellyfin gets media **read-only** in Docker; the arr services own downloads and write into media.

Use shared UID/GID values for the Docker containers, and ensure the NFS export grants that user/group access. There is no `101000` LXC UID translation in a VM.

```shell
PUID=1000
PGID=1000
```

## Stack Overview

### Jellyfin (Docker VM)

- Media server — streams movies, TV, and music to any device
- Config: `/opt/docker/jellyfin`
- Media: `/data/media` (read-only Docker volume mount)

### VPN Gateway (Docker VM)

- **Gluetun** — Docker container that acts as a WireGuard/OpenVPN gateway. qBittorrent uses `network_mode: "service:gluetun"` so all its traffic routes through the VPN tunnel. Only qBittorrent is routed through Gluetun — Sonarr/Radarr stay on the normal network so they can reach indexers and local clients.
- **ProtonVPN Plus confirmed (Aug 2026)** — supports port forwarding (Plus and above), works seamlessly with Gluetun's built-in NAT-PMP support.
- **No sidecar needed** — modern Gluetun handles ProtonVPN port forwarding natively. In the ProtonVPN dashboard, create a WireGuard config with **NAT-PMP (Port Forwarding) ON** on a P2P server, then copy the `PrivateKey` into `.env`. Gluetun writes the forwarded port and pushes it into qBittorrent automatically via `VPN_PORT_FORWARDING_UP_COMMAND`.
- Avoid for seeding: Mullvad (removed port forwarding 2023), NordVPN (never had it)

### Download Client (Docker VM)

- **qBittorrent** — torrent client with built-in search
- Port: 8080 (web UI, exposed via gluetun), 6881 (torrent traffic)
- Downloads to: `/data/downloads/incomplete` → `/data/downloads/complete`
- Runs inside Gluetun's network namespace — no direct internet access, fully isolated behind VPN

### Indexer Manager (Docker VM)

- **Prowlarr** — single proxy for all torrent/usenet indexers. You add indexers here once and they sync automatically to Sonarr, Radarr, Lidarr, and Readarr
- Port: 9696

### Media Managers (Docker VM)

- **Sonarr** — TV show management. Monitors shows you want, searches for new episodes via Prowlarr, sends them to qBittorrent, then renames and organizes them into `/data/media/tv`
- Port: 8989
- **Radarr** — Movie management. Same workflow as Sonarr for movies into `/data/media/movies`
- Port: 7878
- **Lidarr** — Music management. Monitors artists/albums, downloads and organizes into `/data/media/music`
- Port: 8686
- **Readarr** — Book and audiobook management (optional). Same pattern for ebooks/audiobooks into `/data/media/books`
- Port: 8787

### Subtitle Management (optional, Docker VM)

- **Bazarr** — automatically downloads subtitles for your Sonarr/Radarr media libraries
- Port: 6767

### Request Management (optional, Docker VM)

- **Jellyseerr** — clean web UI for users to browse and request movies/TV shows. Integrates with Sonarr/Radarr for downloads and Jellyfin for media visibility
- Port: 5055

### Utility (optional, Docker VM)

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
/data/downloads/complete
      |
      ▼  (Sonarr/Radarr rename + move)
      |
/data/media/{tv,movies,music}
      |
      ├── Jellyfin (Docker VM, read-only) ←── Users
      └── Bazarr (subtitles)

Indexers via Prowlarr ─── Sonarr/Radarr/Lidarr (normal network)
                                  ↕
                          Jellyseerr ←── user requests
```

## docker-compose.yml (structure, Docker VM)

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
  jellyfin:    # port 8096
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

Add widgets after setup (use the Docker VM IP — note `.25` is [[immich|Immich]], not this stack):

- Jellyfin: `http://<docker-vm-ip>:8096`
- Prowlarr: `http://<docker-vm-ip>:9696`
- qBittorrent: `http://<docker-vm-ip>:8080`

## Related

- [[glance]]
- [[immich]]
- [[todo_proxmox]]
- [[todo_reverse_proxy]]
- [[todo_containers]]
- [[goals]]
