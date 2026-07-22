---
title: Proxmox Storage and Resilience
created: 2026-06-16
type: todo
tags:
  - todo
  - proxmox
  - storage
  - backup
---

# Proxmox Storage and Resilience

## Current Host Layout

- Proxmox VE 9.2.3, 24 GB RAM
- `sdb`: 240 GB Kingston SATA SSD; Proxmox OS, swap, and `local-lvm` VM disks
- `sda`: 1 TB Seagate HDD; ext4 `media-storage0`
- `sdc`: 500 GB Toshiba HDD; ext4 `app-data`
- No ZFS pools currently exist

## Priorities

- [ ] Add and configure a UPS with graceful host shutdown.
- [ ] Establish a separate backup destination before making storage changes.
- [ ] Configure daily guest backups and test a full restore.
- [ ] Enable SMART monitoring, scheduled self-tests, and failure notifications for every disk.
- [ ] Restrict the Proxmox management UI to the LAN or VPN and review firewall rules.

## Recommended Hardware Plan

### VM and Container Storage

- [ ] Purchase two matching 1 TB SSDs (SATA or NVMe, according to available ports).
- [ ] Create a mirrored ZFS pool for VM/container disks, e.g. `fast-vm`.
- [ ] Migrate guests from the single 240 GB `local-lvm` SSD to the new mirror.
- [ ] Keep the existing 240 GB SSD for the Proxmox OS initially; add a second boot SSD and mirror it later if high availability of the host is important.

### Bulk Media and Application Data

- [ ] Purchase two matching 1 TB **CMR** HDDs for a mirrored ZFS pool, e.g. `bulk`.
- [ ] Use the pool for media, app data, ISOs/templates, and non-VM bulk storage.
- [ ] Migrate data from `sda` and `sdc` only after a verified backup exists.

### Capacity-only Alternative

- [ ] If adding only one 1 TB HDD, use it as a single-disk ZFS pool only for non-critical data or as an additional backup copy.
- [ ] Do not treat a single-disk ZFS pool as redundant: ZFS adds checksums, compression, and snapshots, but a disk failure still loses the pool.
- [ ] Do not create a mirror from the existing ext4 `sda` disk without first migrating its data: ext4 cannot be converted to ZFS in place.

## ZFS Baseline

- [ ] Create pools using stable `/dev/disk/by-id/` identifiers, never `/dev/sdX` names.
- [ ] Use `ashift=12` and enable `compression=lz4`.
- [ ] Schedule a monthly ZFS scrub and investigate checksum/read/write errors promptly.
- [ ] Keep pools below about 80% capacity.
- [ ] Use snapshots for short-term rollback only; they are not backups.

## Backups and Operations

- [ ] Run Proxmox Backup Server on separate hardware where possible, or use an external disk rotated off-site.
- [ ] Configure retention such as 7 daily, 4 weekly, and 6 monthly backups, sized for actual guest data.
- [ ] Test restoring a VM or container on a regular schedule.
- [ ] Keep Proxmox packages current after confirming backups are healthy.
- [ ] Consider 32 GB+ RAM if running more guests, databases, Plex/Jellyfin transcoding, or heavier ZFS workloads.
- [ ] Consider 2.5 GbE/10 GbE if transferring large media libraries or running network backups.

## Intended End State

```text
Proxmox OS        -> existing 240 GB SSD (mirror later if needed)
VM/container data -> 2 x 1 TB SSD ZFS mirror: fast-vm
Media/app data    -> 2 x 1 TB CMR HDD ZFS mirror: bulk
Backups           -> separate PBS host or rotated/off-site backup disk
```
