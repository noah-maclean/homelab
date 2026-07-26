---
title: Container Install TODO
created: 2026-06-16
type: todo
author: noah
tags:
  - todo
  - containers
  - setup
---

# TODO

- [[immich|immich]]
- [[todo_jellyfin_arr|jellyfin + arr stack]]
- uptime kuma
- try openclaw as well?
- uptime kuma
- [[todo_reverse_proxy|nginx proxy manager or caddy]]

## Open Notebook

- **Project:** [lfnovo/open-notebook](https://github.com/lfnovo/open-notebook) — 36k ⭐, MIT
- **What it is:** Open source, privacy-focused alternative to Google's Notebook LM
- **Stack:** Python/FastAPI + Next.js/React + SurrealDB + LangChain
- **Deploy:** Single `docker compose up -d` — SurrealDB + web app
- **Key features:**
    - 18+ AI providers (OpenAI, Anthropic, Ollama, LM Studio, DeepSeek, etc.)
    - Multi-modal sources: PDFs, video, audio, web pages, Office docs
    - Multi-speaker podcast generation (1–4 speakers)
    - Full REST API, content transformations, MCP integration
    - Local-only with Ollama for free AI
- **Notes:** Needs Docker (use LXC with Docker installed). SurrealDB backend stores notes locally. Encryption key required for API key storage. Web UI on :8502, API on :5055.

## Related

- [[immich]]
- [[todo_jellyfin_arr]]
- [[todo_reverse_proxy]]
- [[goals]]
