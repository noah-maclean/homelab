---
title: Hermes Agent TODO
created: 2026-06-20
updated: 2026-07-26
type: todo
author: noah
tags:
  - todo
  - hermes
  - config
---

# TODO

- [ ] switch to a different provider
- [ ] chatgpt plus via oauth
- [ ] deepseek v4 flash via api/openrouter (offload to pro for harder tasks)
- [ ] opencode go to try different models
- [ ] customise web dashboard

## Hermes LXC SSH Access

- [ ] Generate SSH key pair for Hermes and add public key to Proxmox host's `~/.ssh/authorized_keys`
- [ ] Verify Hermes can SSH into the Proxmox host from its LXC
- [ ] Use `pct enter <CTID>` via the host to manage containers without per-container SSH setup
- [ ] (Optional) If direct SSH into containers is preferred later: install openssh-server, create a non-root user, and add Hermes SSH key per container

Notes:

- Community-script LXCs typically only have root with auto console login and no SSH running
- Preferred approach: SSH into the Proxmox host, then `pct enter <CTID>` — no setup needed per container, survives container rebuilds
- Alternative: direct SSH per container requires installing openssh-server and adding SSH keys to each one

## Related

- [[containers/hermes_agent]]
- [[todo/todo_mcp]]
- [[todo/todo_vault]]
- [[todo/todo_proxmox_storage]]
