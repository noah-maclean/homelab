---
title: Glance Dashboard
status: running
type: container
author: noah, hermes
tags:
  - container
  - dashboard
  - monitoring
---
# Glance

## Setup

- run [Proxmox helper script](https://community-scripts.org/scripts?q=glance) with advanced settings
       - 1 core
       - 512 MB RAM
       - 2 GB HDD
- IP: 192.168.1.22

## Post Creation

- config at `/opt/glance_data/`
- removed search and a bookmark

## Dashboard Widgets

- [[immich|Immich]] added to the home page (07/08/2026)
- [[tailscale|Tailscale]] devices widget
- **Crafty Controller** widget
- **Glance Agent** on the rasppi — server stats
- Icons switched from simple-icons (`si`) to selfh.st (`sh`) for colour images and consistency (07/08/2026) — crafty controller and hermes have no icon on `si`
- In progress: community-widgets, [[todo_jellyfin_arr|Jellyfin stack]], gluetun VPN status, Home Assistant sensor stats, Uptime Kuma

## Related

- [[todo_glance]]
- [[todo_jellyfin_arr]]
- [[todo_reverse_proxy]]
- [[tailscale]]
- [[hermes_agent]]
- [[immich]]
- [[goals]]
