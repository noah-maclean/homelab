---
title: Hermes Agent
status: running
type: container
author: noah, hermes
tags:
  - container
  - ai
  - hermes
---

# Hermes Agent

## Setup

- run Proxmox Helper script
- inside container run `hermes-setup` or `su - hermes` and then `hermes setup`

## SSH & Sudo Access

- User: `hermes`, password set, member of `sudo` group
- SSH on port 22 with password auth (default Debian config)
- Connect: `ssh hermes@192.168.1.23` (LAN) or `ssh hermes@100.94.37.116` (Tailscale)
- Sudo works for any root-level operations inside the container
- Password set and sudo added via `pct enter <CTID>` from the Proxmox host

## Tailscale

- Tailscale installed and joined tailnet as `100.94.37.116`
- `--accept-routes=false` (explicitly set) — otherwise the Tailscale LXC's advertised `192.168.1.0/24` subnet route would be accepted into table 52, routing LAN responses through `tailscale0` instead of `eth0` and breaking local LAN access to Hermes from other containers
- Also set `tailscale set --operator=$USER` so CLI commands don't need `sudo`

## Related

- [[todo_hermes]]
- [[todo_mcp]]
- [[todo_vault]]
- [[glance]]
- [[tailscale]]
