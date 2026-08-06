---
title: Immich
status: running
type: container
author: noah, hermes
tags:
  - container
  - photo
  - media
---

# Immich

## Install History

- **First attempt (17/06/2026):** community-scripts install (2 cores / 6 GB RAM / 20 GB HDD) — dependency download failed while [[pihole|Pi-hole]] was on, install segfaulted (needed more RAM/swap), folder structure got messed up.
- **Reinstall (06/08/2026):** successful. Container 104, 4 cores / 4 GB RAM (6 GB during install) / 24 GB disk, static IP `192.168.1.25`, AMD GPU passthrough auto-detected (video 44 / render 992). Immich 3.1.0.

## Working Setup (06/08/2026)

- **Web UI:** `http://192.168.1.25:2283` (host is `.20`; `.23` is the [[hermes_agent|Hermes]] LXC)
- **Storage:** `/mnt/storage/immich-photos` on the mergerfs pool, bind-mounted at `/mnt/immich-photos` (`pct set 104 -mp0 ...`); `IMMICH_MEDIA_LOCATION=/mnt/immich-photos` in `/opt/immich/.env`; upload symlinks re-pointed to the mount
- **immich user:** UID 999, GID 991 → host-side ownership `100999:100991` (100000 + UID/GID)
- **Managed folders** (`thumbs`, `upload`, `backups`, `library`, `profile`, `encoded-video`) live under `/mnt/immich-photos`; each needs a `.immich` marker file (Immich creates them on startup once it can write)

## Gotcha: bind-mount permissions (host shell vs container)

`immich-web` crash-looped at boot (`Failed to read .../encoded-video/.immich`, ENOENT) because the service user couldn't write the `.immich` markers into the bind mount.

- **Host shell `chown`/`chmod` on `/mnt/storage/immich-photos` works** (real root, correct offset: `100999:100991`, `775`)
- **Inside the container, `chown`/`touch` as root FAILS** — container root maps to host UID 100000, which is "other" on the mount → `Permission denied` / `Operation not permitted`
- **Do file ops inside the container as the service user instead:**
    - `pct exec 104 -- runuser -u immich -- touch /mnt/immich-photos/<folder>/.immich`

## Related

- [[todo_immich]]
- [[todo_containers]]
- [[todo_glance]]
- [[todo_proxmox]]
- [[goals]]
