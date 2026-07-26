---
title: Vault TODO
type: todo
author: hermes
created: 2026-07-22
tags:
  - todo
  - vault
  - obsidian
  - meta
aliases: []
id: todo_vault
---

# TODO

- [ ] check front matters
- [x] enable nvim front matter creation
- [ ] enable [[todo_mcp|proxmox mcp]] to check status of containers and update tags based on that

## CI/CD pipeline

**What is CI/CD?** Continuous Integration / Continuous Delivery — automates checking and deploying your vault code every time you push to GitHub.

- **CI (Continuous Integration)**: auto-runs checks on every push — markdownlint, broken wikilink detection, frontmatter validation
- **CD (Continuous Delivery/Deployment)**: automatically deploys the validated vault to a static site when changes hit `main`

### GitHub Actions for this vault

Simple CI workflow — runs on every push to any branch:

```yaml
# .github/workflows/ci.yml — validates vault quality on every push
name: CI
on: [push]
jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      - run: npm install -g markdownlint-cli2
      - run: markdownlint-cli2 --fix "**/*.md"
```

Additions to consider:

- **Broken wikilink checker**: Python script scanning `[[wikilinks]]` against existing file stems
- **Frontmatter validator**: ensure every `.md` file has valid YAML frontmatter with required fields
- **Auto-commit fixes**: let CI push lint fixes back via `git-auto-commit-action` — useful so linted output doesn't sit in your local checkout

- [ ] Create `.github/workflows/ci.yml` with markdownlint + wikilink checks
- [ ] Optionally add auto-commit of lint fixes

## Publishing vault as a web UI

You can convert the markdown vault into a browsable static site. Best options:

### Quartz (recommended — easiest setup)

[Quartz](https://quartz.jzhao.xyz/) is a static-site generator built for Obsidian vaults. It handles wikilinks, backlinks, tags, graph view, full-text search, LaTeX, and dark mode out of the box.

**How it works internally:**

Quartz is **fully static** — it reads your markdown files and outputs a folder of HTML, CSS, and JS. There is no server, database, sessions, or login system. The output is uploaded to a CDN (GitHub Pages, Cloudflare Pages, Netlify, Vercel) and served as flat files. Every visitor sees exactly the same content — there's no concept of per-user permissions built in.

**Quick start:**

```zsh
# Clone the Quartz starter
git clone https://github.com/jackyzha0/quartz.git ~/quartz-site
cd ~/quartz-site

# Copy your vault content in
cp -r ~/uni-notes/* content/

# Build and preview locally
npx quartz build --serve

# Deploy to GitHub Pages
npx quartz sync
```

Better approach: use the [GitHub template](https://github.com/jackyzha0/quartz/generate) to create a separate repo (e.g., `uni-notes-site`), then push your filtered vault content to it. Keeps the private vault repo separate from the public site repo.

### "Private Pages" in Quartz — what it actually means

Quartz has a feature called [Private Pages](https://quartz.jzhao.xyz/features/private-pages), but **it is NOT authentication**. It means the page is excluded from the build entirely. Three mechanisms:

| Method | What it does |
|--------|-------------|
| `draft: true` in frontmatter | That note is skipped during build |
| `ExplicitPublish` plugin (swap in config) | Only notes with `publish: true` get built |
| `ignorePatterns` in `quartz.config.yaml` | Glob patterns to exclude whole folders/files |

⚠️ **Important caveat**: non-markdown files (images, PDFs, voice memos, etc.) will **still be emitted publicly** even if the markdown referencing them is filtered. You'd need to add them to `ignorePatterns` too.

### Can I add logins / access control?

**Not built into Quartz** — it's a static site generator, so there's no server to check credentials. But you can layer access control on via your hosting provider:

#### Option A: Cloudflare Access (recommended — free for 50 users)

If you host on Cloudflare Pages, you can put [Cloudflare Access](https://www.cloudflare.com/en-gb/zero-trust/access/) in front of it. This gives you:

- Login via Google, GitHub, Microsoft, email one-time code — no passwords to manage
- **Up to 50 users on the free tier**
- Per-URL policies (different rules for different paths)
- Configured entirely in Cloudflare dashboard — no code changes needed
- Also protects non-markdown files (images, PDFs) that `draft: true` would miss

#### Option B: Vercel Password Protection

Vercel has a built-in password gate in Settings → Git → Preview Deployments. On the free tier this works for preview deployments (not production). The Pro plan (£16/mo) extends it to production.

#### Option C: Netlify Basic Auth

Netlify supports basic auth via `_headers` file syntax. Free tier covers branch-based previews only.

#### Option D: DIY reverse proxy (nginx + auth)

Requires a server you control — overkill for a uni vault.

### Recommended setup for this vault

Since the vault is already a **private GitHub repo**, the cleanest approach:

1. Create a separate Quartz repo (via the GitHub template)
2. Use `ExplicitPublish` plugin — only notes with `publish: true` in frontmatter get built
3. Host on **Cloudflare Pages** (free, auto-deploys from GitHub)
4. Enable **Cloudflare Access** (free tier, up to 50 users)
5. Share the link with classmates — they log in with their Google/GitHub account

This gives you: auto-deploy on push → authentication required → only specific notes visible. All for £0.

### Other options

- **Material for MkDocs**: More customisable but needs wikilinks converted to standard markdown links
- **Obsidian Publish**: Official paid option (£20/mo) — zero config, but costs money and still has no access control beyond a shared password
- **Hugo/11ty**: General-purpose static site generators — more flexible but significantly more work to set up

- [ ] Research Quartz vs Material for MkDocs for Uni Notes
- [ ] Decide: public vs authenticated vs fully private
- [ ] If going ahead: create Quartz repo, configure `ExplicitPublish`, deploy to Cloudflare Pages, add Cloudflare Access

## Related

- [[hermes_agent]]
- [[todo_mcp]]
- [[todo_hermes]]
