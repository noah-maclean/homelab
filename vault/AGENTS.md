# Repository Guidelines

## Project Structure & Content Organisation

This repository is a Markdown knowledge vault for the homelab, not an application. Keep durable service notes in `containers/` (for example, `containers/immich.md`), active implementation work in `todo/`, and chronological updates in `logs/` using the `YYYY-MM-DD.md` filename pattern. Use `goals.md` for longer-term priorities. Store a note in the most specific existing area; create a new top-level area only when the content does not fit these categories.

### Todo note organisation

**Splitting rule:** Keep all todo items for a single container in one note, separated by subheadings. Only create a separate todo note when the topic relates to multiple containers (e.g., shared infrastructure, cross-cutting config) or is unrelated to any specific container. Do not split per-container todos into individual files.

**Completion:** Never delete completed items. Mark them with `- [x]` and move them below the active (unchecked) items under the same subheading, keeping the done items grouped at the bottom. If an entire subheading's tasks are all done, mark the heading itself with `✓` (e.g. `## Hermes LXC SSH Access ✓`). This preserves a visible history of what was done without cluttering the active tasks.

**Logging is optional:** Significant completions can also be recorded in `logs/YYYY-MM-DD.md` for narrative context, but this is not required — the archived checkboxes are sufficient on their own.

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
type: container  # container | log | todo | goals
author: <your_name>
tags:
  - container
  - networking
---
```

- **`title`** — human-readable display name. Match the note's `# Heading` or provide a clearer one.
- **`type`** — note category: `container`, `log`, `todo`, or `goals`. Maps to the directory the note lives in. Used for Dataview filtering.
- **`author`** — your identifier (`noah`, `hermes`, `claude-code`, etc.). When you create a new note, set `author:` to yourself. When you edit an existing note's content substantially, add or update `author:` to reflect who made the latest meaningful change.
- **`tags`** — topical tags for Obsidian's tag pane and graph view. Repeat the `type:` value as one tag, then add topic-specific tags (e.g. `networking`, `media`, `proxmox`, `ai`).
- **`status`** (container notes only) — `running`, `not_started`, `paused`, `broken`.
- **`date`** / **`created`** / **`updated`** — ISO dates for logs and timelines.

Always run `npx markdownlint-cli2 --fix "vault/**/*.md"` after adding or editing front matter — YAML formatting must pass linting. The config at `.markdownlint-cli2.jsonc` allows front matter `title:` to coexist with body `#` headings.

## Content Review Guidelines

Treat documentation accuracy as the primary test. Re-read edited commands for quoting, flags, and whether they are safe to paste. For networking, container, or credential guidance, record assumptions and avoid adding live tokens, passwords, private keys, or personal URLs. Update related todo items or logs when a change materially affects current homelab status.

## Commits & Pull Requests

Recent history uses timestamped backup commits, such as `vault backup: 2026-07-08 17:16:10`. For manual commits, use a brief imperative summary scoped to the note, such as `docs: update Tailscale route setup`. Keep each commit focused. Pull requests should explain the operational impact, list files or services affected, link any relevant task, and include screenshots only when documenting rendered layout or visual configuration.

## Linking Conventions

Every note should use **inline wikilinks** where prose mentions another note, plus a **`## Related` section** at the bottom for broader connections. This gives Obsidian's graph view meaningful context while still linking everything topically.

### Inline links (strongest)

Replace bare mentions of other notes with `[[path/to/note|display text]]`:

- In goal/todo lists: `- [ ] [[containers/immich|Immich]]` instead of `- [ ] Immich`
- In prose: `most stay on [[containers/tailscale|Tailscale]]` instead of `most stay on Tailscale`
- In headings: `## [[containers/glance|Glance Dashboard]]`
- Only link the **first** mention of a note per section to avoid clutter.

### Related section (broad connections)

Append after all content, before any trailing blank lines:

```markdown
## Related

- [[containers/hermes_agent]]
- [[todo/todo_jellyfin_arr]]
- [[goals]]
```

Use this for connections that don't have a natural prose anchor (e.g., a container note linking to its todo items, or goals linking to everything it references).

### What to link

| From | Link to |
|------|---------|
| `containers/` notes | Their corresponding `todo/` items, plus related containers |
| `todo/` notes | The `containers/` notes they reference, plus related todos |
| `goals.md` | Every `containers/` and `todo/` note it references |
| Log entries | `containers/` and `todo/` notes mentioned in the day's work |

### Path format

Use **vault-relative paths** — no leading `/`, no `.md` extension:

- `[[containers/glance]]` ✓
- `[[todo/todo_jellyfin_arr]]` ✓
- `[[/vault/containers/glance.md]]` ✗
- `[[glance]]` ✗ (ambiguous with potential future notes)

## Change Documentation

Any time you make or learn about a change to a homelab service (container config, software install, infrastructure change, vault restructuring, etc.):

**If you made the change:** document it in a daily log (`logs/YYYY-MM-DD.md`) immediately — before committing or moving on. Use a clear `##` subheading describing what was done.

**If someone tells you about a change:** ask "Would you like me to log this in today's daily note?" before proceeding.

This ensures the vault stays a complete history regardless of which agent (Hermes, Claude Code, Codex CLI, Cursor, etc.) made the change. The daily consistency sync cron job (`job_id: a44601cc0e9d`) can then propagate details into the relevant `containers/` notes automatically.
