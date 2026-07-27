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

## Remote Access

**LAN IP:** `192.168.1.23` — reachable from anywhere via the [[tailscale|Tailscale]] subnet router (no Tailscale IP needed on your device, your Mac/phone just needs Tailscale connected for the routing to work).

| Tool | Mac | Phone (Termius) |
|------|-----|-----------------|
| **SSH** | `ssh hermes@192.168.1.23 -t herdr` | Host `192.168.1.23`, user `hermes`, start cmd `herdr` |
| **Mosh** (roaming) | `mosh hermes@192.168.1.23 -- herdr` | Termius → protocol MOSH if available |
| **Web Dashboard** | `http://192.168.1.23:9119` in browser | Same URL in phone browser |
| **Discord** | Already connected — no IP needed | Already connected — no IP needed |

All use `192.168.1.23` — the subnet router on the Tailscale LXC routes traffic to the LAN from anywhere.

**Zsh alias for Mac:**

```zsh
alias hermes-lxc='mosh hermes@192.168.1.23 -- herdr'
```

### herdr (replaces tmux)

Herdr 0.7.5 is installed on the LXC with the Hermes integration plugin. Launch with `herdr` after connecting — auto-attaches to your existing session. Detach with `Ctrl+Q`.

## Hermes LXC SSH Access ✓

- [x] (Done) Set password for `hermes` user and add to `sudo` group — SSH + sudo access enabled for remote TUI use
- [x] (Done) Access via LAN IP `192.168.1.23` — subnet router handles remote connections
- [x] (Done) Mosh 1.4.0 installed for roaming sessions (UDP, survives network switches)
- [x] (Done) herdr 0.7.5 installed — replaces tmux for persistent agent workspaces

## Related

- [[hermes_agent]]
- [[todo_mcp]]
- [[todo_vault]]
- [[todo_proxmox]]
