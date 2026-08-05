---
title: Hermes Agent TODO
created: 2026-06-20
updated: 2026-08-06
type: todo
author: noah, hermes
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

## Discord Voice — First-Utterance Hallucination

- [ ] **Debug first-utterance-after-join hallucination** (parked 2026-08-06). First voice utterance after each VC join still transcribes as garbage — e.g. *"I'm going to go to the next one."* — while all follow-ups are clean. Cause: socket warm-up audio has energy, not silence, so the v3 silence-trim doesn't catch it; Whisper hallucinates filler over the junk. Next steps: add a hook to save the first utterance's raw PCM/WAV to disk, say one test phrase, inspect the waveform, then likely require sustained speech (not a single noise spike) before treating audio as an utterance. See [[2026-08-05|08-05 log]] for the full patch history (v1 phrase filter + v3 silence trim, both still live; v2 drop hack removed). Local patches in `tools/voice_mode.py` + `plugins/platforms/discord/adapter.py` must be re-applied after `hermes update`.

## Related

- [[hermes_agent]]
- [[todo_mcp]]
- [[todo_vault]]
- [[todo_proxmox]]
