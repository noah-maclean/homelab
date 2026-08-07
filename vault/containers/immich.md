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

- **Web UI:** `http://192.168.1.25:2283`
- **Storage:** `/mnt/storage/immich-photos` on the mergerfs pool, bind-mounted at `/mnt/immich-photos` (`pct set 104 -mp0 ...`); `IMMICH_MEDIA_LOCATION=/mnt/immich-photos` in `/opt/immich/.env`; upload symlinks re-pointed to the mount
- **immich user:** UID 999, GID 991 → host-side ownership `100999:100991` (100000 + UID/GID)
- **Managed folders** (`thumbs`, `upload`, `backups`, `library`, `profile`, `encoded-video`) live under `/mnt/immich-photos`; each needs a `.immich` marker file (Immich creates them on startup once it can write)

## Install Instructions (verified 06/08/2026)

Simple one-command-at-a-time procedure. **Host shell** = Proxmox host (root@192.168.1.20) — real root, can chown bind mounts. **Container** = inside the LXC: run `pct exec 104 -- <cmd>` from the host, or `pct enter 104`. Note: `pct exec 104 -- ...` *executes inside the container* — container root is NOT privileged over the bind mount.

1. **Turn Pi-hole OFF** — dependency downloads fail while it's on. (Pi-hole is the [[pihole|rasppi]] at 192.168.1.10)
2. **Host shell:** run the community-scripts Immich installer (Advanced) — creates container 104, 4 cores / 6 GB RAM / 24 GB disk, IP as chosen, AMD GPU passthrough, compiles photo libraries (15 min–2 h). *Pi-hole must be off for this.*
3. **Host shell:** `pct set 104 -memory 4096` — drop RAM from 6 GB to 4 GB now the build is done. (**or do in web ui LXC settings**)
4. **Host shell:** `pct set 104 -net0 name=eth0,bridge=vmbr0,gw=192.168.1.254,ip=192.168.1.25/24,type=veth` — set the static IP to `192.168.1.25` (**or do in web ui LXC settings**)
5. **Host shell:** `mkdir -p /mnt/storage/immich-photos` — create the storage folder on the mergerfs pool.
6. **Host shell:** `chown -R 100999:100991 /mnt/storage/immich-photos` — give it to the container's immich user (100000 + container UID/GID = 999/991). *Do this on the host — inside the container it fails with "Operation not permitted".*
7. **Host shell:** `pct set 104 -mp0 /mnt/storage/immich-photos,mp=/mnt/immich-photos` — bind-mount the pool into the container.
8. **Container:** `sed -i 's|^IMMICH_MEDIA_LOCATION=.*|IMMICH_MEDIA_LOCATION=/mnt/immich-photos|' /opt/immich/.env` — point Immich's media location at the mount (via `pct exec 104 -- sed -i ...`).
9. **Container:** `rm /opt/immich/app/upload /opt/immich/app/machine-learning/upload && ln -s /mnt/immich-photos /opt/immich/app/upload && ln -s /mnt/immich-photos /opt/immich/app/machine-learning/upload` — re-point the upload symlinks at the mount (via `pct exec 104 -- rm ... && pct exec 104 -- ln -s ...`).
10. **Host shell:** `mkdir -p /mnt/storage/immich-photos/{thumbs,upload,backups,library,profile,encoded-video} && chown -R 100999:100991 /mnt/storage/immich-photos && chmod -R 775 /mnt/storage/immich-photos` — pre-create Immich's managed folders with correct ownership. *On the host, as real root — container root can't.*
11. **Container, as the immich user (NOT root):** `runuser -u immich -- bash -c 'for d in thumbs upload backups library profile encoded-video; do touch /mnt/immich-photos/$d/.immich; done'` — create the `.immich` marker files (via `pct exec 104 -- runuser -u immich -- ...`). This is the crash-loop fix — as root inside the container it fails with "Permission denied"; as immich it works.
12. **Container:** `systemctl restart immich-web immich-ml` — start both services (via `pct exec 104 -- systemctl restart immich-web immich-ml`).
13. **Verify:** open `http://192.168.1.25:2283`, upload a test photo, confirm it lands in `/mnt/storage/immich-photos`.
14. **Turn Pi-hole back ON.**

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
