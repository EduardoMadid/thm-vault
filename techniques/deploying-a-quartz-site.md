---
title: deploying-a-quartz-site
tags: [devops, git, github-actions, ci-cd, ssh]
---

# Deploying a Quartz site with Git & GitHub Actions

## In plain terms
This is the story of taking a notes site from "works on my laptop" to "live on the internet, updating itself." Every time I push a change, a robot on GitHub's servers rebuilds the whole site and publishes it — no manual steps. Getting there meant learning Git properly, setting up secure authentication, and writing the automation that glues two separate repos into one website.

## The architecture
Two repositories, one website:
- **`cyber-notes`** — the Quartz engine + theme + the boot landing page
- **`thm-vault`** — the actual content (writeups, techniques), kept separate so notes aren't tangled with the engine

A GitHub Action in `cyber-notes` clones `thm-vault` at build time, drops it into `content/`, builds, and publishes to GitHub Pages.

## Git — what I actually used
- The mental model: working directory → staging (`git add`) → repo (`git commit`). History is a series of snapshots.
- Adopting a clone: removed the upstream `.git` (`rm -rf .git`) and ran a fresh `git init` so Quartz became *my* repo, not a fork of jackyzha0's.
- `.gitignore` discipline — never version `node_modules/`, `public/` (build output), or app config (`.obsidian/`). All reconstructible.
- Two remotes, two repos, `git remote add origin`, `git push -u origin main`.

## SSH auth
- Generated a dedicated key pair (`ed25519`) just for GitHub — isolation: if it leaks, revoke only this one.
- Public/private key model — same asymmetric crypto idea as DPAPI: private stays local and proves identity, public is shared and only verifies.
- Mapped the key to the host in `~/.ssh/config` (`IdentityFile`, `IdentitiesOnly yes`), tested with `ssh -T git@github.com`.

## The GitHub Action (the core)
The workflow (`.github/workflows/deploy.yaml`) does, on every push to `main`:
1. Checkout the repo
2. **Clone `thm-vault` into `content/`** — this is what joins the two repos
3. `npm ci` — install dependencies
4. `npx quartz build` — generate the static site into `public/`
5. Upload + deploy to GitHub Pages

Key concepts learned:
- **Triggers** (`on: push: branches: [main]`), **jobs**, **steps**, **permissions** (pages needs `write`).
- **YAML indentation matters** — a `run: |` block's commands must sit deeper than the `run:` key. Validated locally with `python -c "import yaml; ..."` before burning a CI run.
- **Orchestrating two repos** — the build pulls content from a second repo, which is the whole trick of the "separate repos" architecture.

## Gotchas I hit
- `baseUrl` must be set to the real Pages URL (`user.github.io/repo`) — no `https://`, no trailing slash. RSS and links break otherwise.
- Static assets land under `/static/...`, not the root — the boot landing ended up at `/cyber-notes/static/boot/`, not `/cyber-notes/boot/`.
- A push to the content repo doesn't rebuild the site — had to trigger the engine repo's Action (empty commit or re-run).
- `fish` doesn't do bash heredocs (`<< EOF`) — use `printf` instead.

## 🧠 Insight
The "separate repos" approach costs you a more complex Action but keeps content clean from tooling. The Action *is* the glue — everything hard lives in ~40 lines of YAML, and once it's right, the whole thing runs itself.