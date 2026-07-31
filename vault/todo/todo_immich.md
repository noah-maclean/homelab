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

## LXC Setup

- [ ] Turn Pi-hole OFF before install (dependency downloads fail with it on)
- [ ] Run community-scripts installer (Advanced): 4 cores / 6 GB RAM / 30 GB disk, IP 192.168.1.24 (NOT .23 — [[hermes_agent|Hermes]] LXC has it)
- [ ] Reduce RAM to 4 GB after install
- [ ] Host: `mkdir -p /mnt/storage/immich-photos`
- [ ] Host: get container UID and chown the host dir
       - `pct exec <VMID> -- id -u immich` (e.g. 999)
       - `chown -R 100999:100999 /mnt/storage/immich-photos` (100000 + container UID)
- [ ] Host: bind mount the mergerfs pool
       - `pct set <VMID> -mp0 /mnt/storage/immich-photos,mp=/mnt/immich-photos`
- [ ] Container: edit `/opt/immich/.env` → `IMMICH_MEDIA_LOCATION=/mnt/immich-photos`
- [ ] Container: re-point symlinks to the mount (installer only links `/opt/immich/upload`)
       - `rm /opt/immich/app/upload /opt/immich/app/machine-learning/upload`
       - `ln -s /mnt/immich-photos /opt/immich/app/upload`
       - `ln -s /mnt/immich-photos /opt/immich/app/machine-learning/upload`
- [ ] Container: `systemctl restart immich-web immich-ml`
- [ ] Verify uploads land on the host: `/mnt/storage/immich-photos`

## Glance Widget

- [ ] Add Immich widget to [[glance]] (from [[todo_glance]])

## Related

- [[todo_containers]]
- [[todo_glance]]
- [[goals]]
