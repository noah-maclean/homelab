# Repository Guidelines

## Project Structure & Content Organisation

This repository is a Markdown knowledge vault for the homelab, not an application. Keep durable service notes in `containers/` (for example, `containers/immich.md`), active implementation work in `todo/`, and chronological updates in `logs/` using the `YYYY-MM-DD.md` filename pattern. Use `goals.md` for longer-term priorities. Store a note in the most specific existing area; create a new top-level area only when the content does not fit these categories.

## Development and Validation

There is no build system, test suite, or runtime command in this repository. Validate changes before committing by reviewing the rendered Markdown in your editor and checking links, command syntax, and heading hierarchy. Useful checks include:

```shell
git diff --check       # find whitespace errors
git diff -- containers/ # review service-note changes
```

Do not run operational commands copied from notes against infrastructure unless the task explicitly calls for it.

## Formatting

All Markdown in this vault must pass `markdownlint-cli2` with zero errors before committing. The project config is at `.markdownlint-cli2.jsonc` in the repo root.

To check and auto-fix:

```shell
npx markdownlint-cli2 --fix "vault/**/*.md"
```

Run this before every commit. This keeps the entire vault consistently formatted regardless of which tool wrote the file.

## Writing Style & Naming

Write concise, practical Markdown. Use ATX headings (`#`, `##`), fenced code blocks with a language such as `shell`, and `-` for lists. Preserve the local style of the note being edited, including its indentation. Prefer short, descriptive lowercase filenames with underscores for topical notes (for example, `todo_jellyfin_arr.md`); use ISO dates for logs. Keep commands copyable, include required context such as IPs or prerequisites, and clearly label secrets or values that must not be committed.

## Front Matter & Attribution

Every vault note MUST have YAML front matter between `---` delimiters with the following fields:

```yaml
---
title: Note Title
author: <your_name>
tags:
  - relevant
  - tags
---
```

- **`title`** — human-readable display name. Match the note's `# Heading` or provide a clearer one.
- **`author`** — your identifier (`noah`, `hermes`, `claude-code`, etc.). When you create a new note, set `author:` to yourself. When you edit an existing note's content substantially, add or update `author:` to reflect who made the latest meaningful change.
- **`tags`** — at minimum the note type (`container`, `log`, `todo`, `goals`). Add topical tags (e.g. `networking`, `media`, `proxmox`) for queryability.
- **`status`** (container notes only) — `running`, `not_started`, `paused`, `broken`.
- **`date`** / **`created`** / **`updated`** — ISO dates for logs and timelines.

Always run `npx markdownlint-cli2 --fix "vault/**/*.md"` after adding or editing front matter — YAML formatting must pass linting. The config at `.markdownlint-cli2.jsonc` allows front matter `title:` to coexist with body `#` headings.

## Content Review Guidelines

Treat documentation accuracy as the primary test. Re-read edited commands for quoting, flags, and whether they are safe to paste. For networking, container, or credential guidance, record assumptions and avoid adding live tokens, passwords, private keys, or personal URLs. Update related todo items or logs when a change materially affects current homelab status.

## Commits & Pull Requests

Recent history uses timestamped backup commits, such as `vault backup: 2026-07-08 17:16:10`. For manual commits, use a brief imperative summary scoped to the note, such as `docs: update Tailscale route setup`. Keep each commit focused. Pull requests should explain the operational impact, list files or services affected, link any relevant task, and include screenshots only when documenting rendered layout or visual configuration.
