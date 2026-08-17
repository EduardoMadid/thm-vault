---
title: deploying-a-quartz-site
tags:
  - devops
  - git
  - github-actions
  - ci-cd
  - ssh
  - bash
  - automation
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

The deploy script is the same idea one layer up: the Action automates the *build*, the script automates the *push*. Both replace a fragile manual ritual with something that runs itself but I kept a confirmation prompt in the script, because fully automating a `git push` across two repos is exactly how you publish a mistake at 2am.

## Automating the deploy

Two repos means two `add`/`commit`/`push` cycles every time I publish, plus remembering that a content push needs a manual engine rebuild. I wrote a bash script to collapse all of that into one command:

```bash
./deploy.sh "commit message"            # normal: commits real changes in both repos
./deploy.sh "commit message" --rebuild  # quartz gets an empty commit to force a rebuild
```

The script deploys the vault first (content), then the quartz repo (engine), asking for confirmation before each push so I can eyeball `git status` before anything goes up. The `--rebuild` flag switches the quartz side to an empty commit — the exact trick from the gotcha above, now automated.

Design decisions worth noting:
- **Guard clause first** — bails out immediately if no commit message is passed, so nothing half-runs.
- **A `--rebuild` boolean** captured once at the top, used later — separates *deciding* from *acting*.
- **`commit_if_changes`** — checks `git diff --quiet` before committing so an empty "nothing to commit" doesn't abort the run; only pushes if a commit actually happened. The `--rebuild` branch deliberately skips this, since an empty commit is the whole point there.
- **Confirmation prompts** — `git status` + a `read` before each push. The trade-off for automation is losing the "inventory before touching" ritual, so the prompt puts it back.
- **`cd ... || { echo; exit; }`** — if a repo folder is missing, it says so and stops instead of running git in the wrong place.

Testing discipline: I swapped every `git push` for `echo git push` first, ran both modes to watch the flow without publishing anything, then removed the echoes once it behaved.