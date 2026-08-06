---
title: Immich
status: not_started
type: container
author: noah, hermes
tags:
  - container
  - photo
  - media
---
# Immich

## Install History

- **First attempt (17/06/2026):** community-scripts install (2 cores / 6 GB RAM / 20 GB HDD) — dependency download failed while [[pihole|Pi-hole]] was on, install segfaulted (needed more RAM/swap), and the folder structure got messed up. Second try that day used 6 GB RAM with Pi-hole off, bind-mounted `/mnt/storage/immich-photos/upload` → `/mnt/immich-photos`, set `IMMICH_MEDIA_LOCATION` in `.env`, and re-pointed the upload symlinks. The LXC still isn't running — see the re-setup plan below.

## LXC Re-setup Plan (31/07/2026)

- Full verified procedure in [[todo_immich]], checked against the current community-scripts installer (`install/immich-install.sh` + `ct/immich.sh`)
- Installer now creates the upload symlinks itself (`ln -s $UPLOAD_DIR app/upload` + machine-learning)
- `immich` user is now a **system user** (`useradd -r`) — host-side chown is `100000 + container UID` (e.g. `100999`), not the old `101000`
- Install is a full source build (libjxl/libheif/libraw/ImageMagick/libvips) — needs **6 GB RAM**, Pi-hole off
- Container IP: `192.168.1.25` — `.23` is taken by the [[hermes_agent|Hermes]] LXC
- Storage: `/mnt/storage/immich-photos` on the mergerfs pool, bind-mounted into the container

## Related

- [[todo_immich]]
- [[todo_containers]]
- [[todo_glance]]
- [[todo_proxmox]]
- [[goals]]
