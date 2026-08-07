---
title: Jellyfin + Arr Stack
status: not_started
type: container
author: hermes
tags:
  - container
  - media
  - arr
---

# Jellyfin + Arr Stack

Planned media stack — not yet deployed. Full implementation plan in [[todo_jellyfin_arr]].

## Current Plan (2026-08-06)

**One Docker VM** running the complete stack in a single Compose project:

- Type: Debian or Ubuntu Server VM with Docker Engine + Compose
- Resources: 4 cores / 4096–6144 MB RAM / 32 GB system disk
- Services: Jellyfin, Gluetun (ProtonVPN WireGuard), qBittorrent, Prowlarr, Sonarr, Radarr (+ optional Lidarr, Readarr, Bazarr, Jellyseerr, Watchtower, FlareSolverr)
- Storage: NFS mount of `/mnt/storage` into the VM at `/data` (a VM can't use `pct` bind mounts); Jellyfin gets media read-only
- VPN: ProtonVPN Plus confirmed — Gluetun's built-in NAT-PMP handles port forwarding, so no `qsticky` sidecar is needed
- Glance widgets after setup: Jellyfin `:8096`, Prowlarr `:9696`, qBittorrent `:8080` (IP TBD)

## Decision History

- **Jul 2026:** single privileged LXC with Docker (superseded — the runc 1.2+ breakage that motivated it is fixed on PVE 9)
- **2026-08-06 (21:19):** two-LXC plan — Jellyfin native (`jellyfin.sh`) + arr stack in Docker on an unprivileged LXC (`docker.sh`) — researched, logged in [[2026-08-06|08-06 log]], and written to the todo
- **2026-08-06 (21:35):** todo revised to a **single Docker VM** for the whole stack — this is the current plan of record; the two-LXC variant is retained for history

## Related

- [[todo_jellyfin_arr]]
- [[glance]]
- [[immich]]
- [[todo_proxmox]]
- [[todo_reverse_proxy]]
- [[goals]]
