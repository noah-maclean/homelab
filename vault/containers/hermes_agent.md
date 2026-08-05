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

## herdr

Herdr 0.7.5 is installed on the LXC as the agent session manager (replaces tmux):

- **Install**: `curl -fsSL https://herdr.dev/install.sh | sh` into `~/.local/bin/herdr`
- **Integration**: `herdr integration install hermes` enabled the v3 lifecycle plugin at `~/.hermes/plugins/herdr-agent-state/` — gives herdr proper idle/working/blocked state tracking and session management
- **PATH**: `~/.local/bin` added to `~/.bashrc` above the interactive guard so herdr panes can find both `hermes` and `herdr`
- **Dashboard**: runs on port 9119 (PID 2808)
- **Launch**: connect via SSH/Mosh then run `herdr` — auto-attaches to existing session. Detach with `Ctrl+Q`.

## Changes

### 2026-08-01 — SOUL.md

- Replaced the default Hermes identity (`~/.hermes/SOUL.md`) with a custom soul adapted from Noah's "Robespierre" template — default backed up to `~/.hermes/SOUL.md.bak.20260801_*`
- New soul covers: role (Hermes in Proxmox LXC 2C/4GB/20GB, user is Noah), tone (brief, direct, lead with the answer, stop at decisions/errors), and compaction behaviour
- Added clause: all server information lives in the homelab vault (`~/homelab`) — hardware, services, configs, changes, todos — and service/config changes get documented in the daily log
- Added clause (amended same day): always ask for clarification when needed — if something cannot be done, or documentation Noah should know the location of cannot be found, ask instead of guessing
- Takes effect from the next session (SOUL.md loads at session start)

### 2026-07-27 — Desktop App & Vision Model

- **Desktop app**: Set up the Hermes Desktop app on Mac (Noah's daily-driver machine). Connected to the remote Hermes LXC instance — the desktop app runs locally on Mac and talks to the remote Hermes backend over the network, giving a native chat GUI instead of terminal-only access.
- **Raycast launcher**: Created a Raycast script command at `~/.config/raycast/scripts/hermes.sh` that runs `hermes desktop` to bypass the app's first-run provisioning reinstall issue. Launches straight into the remote backend session from Raycast/Spotlight.
- **Vision model**: Configured the vision provider to use the Google AI Studio free API key. The vision model is set to `gemini-flash-latest` (dynamic alias that currently resolves to **Gemini 3.6 Flash**). This keeps vision fully on the free tier with no risk of charges — the free tier covers input, output, and vision at no cost with rate limits (10+ RPM / 250+ RPD) that are more than sufficient for occasional image analysis use.

### 2026-07-26 — Tailscale `--accept-routes` Fix

- See [[2026-07-27|log entry for 2026-07-27]] for details.

### 2026-07-18 — Update Checker Cron

- Daily cron job (`f4ae53358894`) checks for Hermes Agent updates at 10:00 and delivers a summary to the Discord `#updates` channel — see [[2026-07-18|log entry for 2026-07-18]]
- Script: `~/.hermes/scripts/hermes-update-check.py`; originally `no_agent=True` (zero tokens per tick)
- Notifies **once per release** via state file `~/.hermes/scripts/.hermes-update-state.json`, not daily
- Upgraded to **agent mode**: the script collects structured release data; the cron agent adds an ⚠️ **Impact on you** section covering breaking changes affecting `config.yaml`/`.env`/skills, Discord/Telegram interaction changes, relevant new features, and anything needed before `hermes update`
- Silence preserved on quiet days via the cron `[SILENT]` marker

## Related

- [[todo_hermes]]
- [[todo_mcp]]
- [[todo_vault]]
- [[glance]]
- [[tailscale]]
