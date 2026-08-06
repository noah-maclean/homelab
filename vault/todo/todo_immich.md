---
title: Immich TODO
created: 2026-07-31
type: todo
author: hermes
tags:
  - todo
  - immich
  - media
---

# TODO

## LXC Setup ✓

- [x] Turn Pi-hole OFF before install (dependency downloads fail with it on)
- [x] Run community-scripts installer (Advanced): 4 cores / 6 GB RAM / 24 GB disk, container 104
- [x] Static IP `192.168.1.25` (host is `.20`; `.23` is [[hermes_agent|Hermes]] LXC — the original plan said `.24`, but `.25` was used)
- [x] Reduce RAM to 4 GB after install
- [x] Host: `mkdir -p /mnt/storage/immich-photos`
- [x] Host: chown the host dir — `chown -R 100999:100991 /mnt/storage/immich-photos` (immich UID 999 / GID 991)
- [x] Host: bind mount the mergerfs pool — `pct set 104 -mp0 /mnt/storage/immich-photos,mp=/mnt/immich-photos`
- [x] Container: edit `/opt/immich/.env` → `IMMICH_MEDIA_LOCATION=/mnt/immich-photos`
- [x] Container: re-point symlinks to the mount (`app/upload`, `machine-learning/upload` → `/mnt/immich-photos`)
- [x] Container: `systemctl restart immich-web immich-ml`
- [x] Fix `immich-web` crash loop — create `.immich` markers as the **immich** user (`runuser -u immich -- touch ...`), not as root (see [[immich]] gotcha)
- [x] Verify uploads land on the host: `/mnt/storage/immich-photos`

## Glance Widget

- [ ] Add Immich widget to [[glance]] (from [[todo_glance]])

## Related

- [[todo_containers]]
- [[todo_glance]]
- [[goals]]
