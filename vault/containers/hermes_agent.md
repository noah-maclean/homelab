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
- Connect: `ssh hermes@192.168.1.23`
- Sudo works for any root-level operations inside the container
- Password set and sudo added via `pct enter <CTID>` from the Proxmox host

## Related

- [[todo_hermes]]
- [[todo_mcp]]
- [[todo_vault]]
- [[glance]]
- [[tailscale]]
