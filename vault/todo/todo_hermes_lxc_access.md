---
title: Hermes LXC SSH Access
created: 2026-07-26
type: todo
author: hermes
tags:
  - todo
  - hermes
  - proxmox
  - ssh
---

# TODO

- [ ] Generate SSH key pair for Hermes and add public key to Proxmox host's `~/.ssh/authorized_keys`
- [ ] Verify I can SSH into the Proxmox host from this Hermes LXC
- [ ] Use `pct enter <CTID>` via the host to manage containers without per-container SSH setup
- [ ] (Optional) If direct SSH into containers is preferred later: install openssh-server, create a non-root user, and add Hermes SSH key per container

Notes:

- Community-script LXCs typically only have root with auto console login and no SSH running
- Preferred approach: SSH into the Proxmox host, then `pct enter <CTID>` — no setup needed per container, survives container rebuilds
- Alternative: direct SSH per container requires installing openssh-server and adding SSH keys to each one

## Related

- [[containers/hermes_agent]]
- [[todo/todo_containers]]
- [[todo/todo_proxmox_storage]]
