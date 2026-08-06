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

### 2026-08-06 — Telegram channel removed

- Removed the Telegram gateway channel — unused; Discord is the only messaging platform in active use. Verified no cron jobs or features deliver to Telegram.
- `~/.hermes/.env` — commented out `TELEGRAM_BOT_TOKEN`, `TELEGRAM_ALLOWED_USERS`, `TELEGRAM_HOME_CHANNEL` (backup: `~/.hermes/.env.bak-telegram-20260806`, token intact, reversible)
- `~/.hermes/config.yaml` — removed the `telegram:` section via `hermes config unset telegram`
- Gateway not restarted during the change; the running process keeps the bot connection until the next `hermes-gateway.service` restart, then it drops permanently

### 2026-08-06 — Discord voice: first-utterance debugging parked

- First utterance after each VC join still transcribes as garbage (e.g. *"I'm going to go to the next one."*) after three patch attempts. Root cause: socket warm-up audio has energy, not silence, so the v3 silence-trim can't catch it.
- **Parked** — next steps logged in [[todo_hermes|Hermes TODO]]: save first-utterance raw audio, inspect waveform, likely require sustained speech before treating audio as an utterance
- Local patches (v1 phrase filter + v3 silence trim) remain live but must be re-applied after `hermes update`

### 2026-08-05 — Discord voice: ambient bed off + local hallucination patches

- Ambient bed disabled (`hermes config set discord.voice_fx.ambient_enabled false`) — the synthesised pad loop made the bot hear its own audio and hallucinate lyrics; supersedes the ambient enable in the section below
- LOCAL PATCH (`tools/voice_mode.py`): `_repeated_phrase_hallucination()` flags any 4+ word contiguous sequence appearing 2+ times; repeat-filler regex now requires 3+ filler words; verified against 18 test cases
- LOCAL PATCH v3 (`plugins/platforms/discord/adapter.py`): `VoiceReceiver.trim_leading_silence()` cuts silent prefixes before STT (20ms RMS windows, threshold 200, 80ms lead-in) — replaces the rejected v2 drop-first-utterance hack
- ⚠️ Both patches are local — must be re-applied after `hermes update`; need gateway restart

### 2026-08-05 — Groq STT live

- `stt.provider` switched `local` → `groq` with `stt.groq.model: whisper-large-v3-turbo` (free tier, ~6x faster than large-v3, near-identical accuracy); `GROQ_API_KEY` added to `~/.hermes/.env` (chmod 600) and verified via `GET /v1/models`
- Local STT model had been dropped `base` → `tiny` while on faster-whisper; Groq now supersedes it for Discord voice
- Takes effect after gateway restart (`/restart` or `systemctl --user restart hermes-gateway`); TTS stays on Edge (free, no key)

### 2026-08-05 — TUI npm install failure fixed

- `hermes --tui` failed with "npm install failed": repo `.npmrc` `engine-strict=true` requires `npm <11.10.0 || >=11.17.0`, system npm was 11.11.0
- Installed user-local `npm@11.17.0` (`--prefix ~/.npm-global`), added `~/.npm-global/bin` to PATH in `~/.bashrc` — takes PATH precedence over system npm for future updates

### 2026-08-05 — Discord Voice: acks + ambient bed

- Enabled `discord.voice_fx` (Discord-only): `enabled: true`, `ack_enabled: true` — bot speaks a canned phrase ("Let me look into that.") before tool calls during VC conversations, plus a low-volume ambient "thinking" bed while tools run
- Reason: bot previously only spoke the final reply; tool-call work was silent in VC
- Needs gateway restart + VC re-join to take effect; see [[2026-08-05|log entry]]

### 2026-08-01 — SOUL.md

- Replaced the default Hermes identity (`~/.hermes/SOUL.md`) with a custom soul adapted from the "Robespierre" template — default backed up to `~/.hermes/SOUL.md.bak.20260801_*`
- New soul covers: role (Hermes in Proxmox LXC 2C/4GB/20GB, user is the homelab owner), tone (brief, direct, lead with the answer, stop at decisions/errors), and compaction behaviour
- Added clause: all server information lives in the homelab vault (`~/homelab`) — hardware, services, configs, changes, todos — and service/config changes get documented in the daily log
- Added clause (amended same day): always ask for clarification when needed — if something cannot be done, or documentation the user should know the location of cannot be found, ask instead of guessing
- Takes effect from the next session (SOUL.md loads at session start)

### 2026-07-27 — Desktop App & Vision Model

- **Desktop app**: Set up the Hermes Desktop app on Mac (the daily-driver machine). Connected to the remote Hermes LXC instance — the desktop app runs locally on Mac and talks to the remote Hermes backend over the network, giving a native chat GUI instead of terminal-only access.
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
