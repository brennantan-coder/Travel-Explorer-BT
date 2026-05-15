---
description: Safely push code to GitHub, ensure README exists, enable GitHub Pages via Actions, update repo About — with secret-scanning safeguards.
argument-hint: "[optional commit message]"
---

You are running the `/gitpush` workflow for this repository. Follow every step in order. Do not skip the safety checks. The commit message argument (if provided) is: `$ARGUMENTS`.

## Step 1 — Confirm repo state

Run in parallel:
- `git status` (no `-uall`)
- `git remote -v`
- `git log --oneline -5`
- `git branch --show-current`

If there is **no remote**, stop and ask the user for the GitHub repo URL before continuing. If the current branch is not `main` or `master`, ask the user whether to push the current branch or switch.

## Step 2 — Secret & sensitive-file scan (BLOCKING)

This step is non-negotiable. Do not proceed to commit until it passes.

1. Ensure a `.gitignore` exists at the repo root. If it does not, create one covering at minimum:
   ```
   # Secrets / env
   .env
   .env.*
   !.env.example
   *.pem
   *.key
   *.p12
   *.pfx
   credentials.json
   secrets.json
   service-account*.json
   .aws/
   .ssh/

   # Local tooling / editor
   .vscode/
   .idea/
   *.swp
   .DS_Store
   Thumbs.db

   # Claude local artifacts (NOT skills/commands — those are committed)
   .claude/settings.local.json
   .claude (temp)/
   ".claude (temp)"
   ```
   Preserve any existing `.gitignore` entries — append, do not overwrite.

2. List files about to be committed: `git status --porcelain` plus `git ls-files --others --exclude-standard`. For each candidate file:
   - **Reject by filename** if it matches: `.env*` (except `.env.example`), `*.pem`, `*.key`, `*.pfx`, `*.p12`, `id_rsa*`, `*.crt` (private), `credentials*.json`, `secrets*.json`, `service-account*.json`, `*.kdbx`, `*token*.txt`.
   - **Content scan** every staged/about-to-be-staged text file (skip binaries and files >1 MB) for these patterns using Grep:
     - `AKIA[0-9A-Z]{16}` (AWS access key)
     - `aws_secret_access_key\s*=`
     - `sk-[A-Za-z0-9]{20,}` (OpenAI/Anthropic-style keys)
     - `ghp_[A-Za-z0-9]{36}`, `gho_[A-Za-z0-9]{36}`, `github_pat_[A-Za-z0-9_]{82}`
     - `xox[baprs]-[A-Za-z0-9-]{10,}` (Slack)
     - `-----BEGIN (RSA |EC |OPENSSH |DSA |PGP )?PRIVATE KEY-----`
     - `(?i)(api[_-]?key|api[_-]?secret|password|passwd|secret|token)\s*[:=]\s*['"][^'"$\{<]{8,}['"]`
     - `firebase.*apiKey\s*:\s*['"][^'"]+['"]`
   - If any hit is found, **stop**. Show the user the file, line, and matched pattern, and ask whether to (a) remove the file, (b) move the value to an env var / secret, or (c) acknowledge and override (only if it's a public/test value).

3. Check existing tracked files with the same scan — if a secret was committed in a prior commit, warn the user; do not silently push.

## Step 3 — README

If `README.md` does not exist at the repo root, generate one. Inspect the repo first (look at `index.html`, `package.json`, top-level source files) and produce a README with:
- Project title (infer from repo name / `<title>` / package name)
- One-paragraph description of what the project does
- "Live Demo" section linking to the GitHub Pages URL (format: `https://<owner>.github.io/<repo>/`)
- "Local development" / "Usage" with the actual commands needed
- "Deployment" noting it auto-deploys via GitHub Actions on push to the default branch
- "License" if a LICENSE file is present

If `README.md` already exists, leave it alone unless the user explicitly asked to regenerate it.

## Step 4 — GitHub Pages workflow

Ensure `.github/workflows/deploy.yml` exists and is committed. If it is missing (deleted, never created), create it with this content (suitable for static sites served from repo root; if the repo has a build step like `npm run build`, adapt accordingly and confirm with the user):

```yaml
name: Deploy to GitHub Pages

on:
  push:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

jobs:
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Pages
        uses: actions/configure-pages@v5

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: '.'

      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

Remind the user that Pages must be enabled in the repo settings with **Source = "GitHub Actions"** the first time. Provide the direct settings URL: `https://github.com/<owner>/<repo>/settings/pages`.

## Step 5 — Commit & push

1. Stage explicit paths (never `git add -A` or `git add .`) — list each file you intend to commit and add them by name.
2. Commit. Use the user's `$ARGUMENTS` as the commit message if provided; otherwise generate a concise message describing the actual changes (Conventional Commits style preferred — e.g., `chore: add deploy workflow and README`). Append the Claude co-author trailer per repo convention.
3. Push to the current branch's upstream. If no upstream is set, `git push -u origin <branch>`.
4. Run `git status` after the push to confirm a clean tree.

## Step 6 — Update repo "About"

Try this in order; stop at the first that works:

1. **`gh` CLI** (if available): `gh repo edit --description "<one-line description>" --homepage "https://<owner>.github.io/<repo>/" --add-topic <topic1> --add-topic <topic2>`. Derive description and topics from the README and project content.
2. **GitHub API via PowerShell** (if a `GH_TOKEN` or `GITHUB_TOKEN` is in `$env:`):
   ```powershell
   $body = @{ description = "..."; homepage = "..."; topics = @("...") } | ConvertTo-Json
   Invoke-RestMethod -Method Patch -Uri "https://api.github.com/repos/<owner>/<repo>" `
     -Headers @{ Authorization = "Bearer $env:GH_TOKEN"; "X-GitHub-Api-Version" = "2022-11-28" } `
     -Body $body -ContentType "application/json"
   ```
3. **Manual fallback**: print the description, homepage URL, and suggested topics, and tell the user to paste them into `https://github.com/<owner>/<repo>` → ⚙ Settings icon next to "About".

## Step 7 — Report back

Output a short summary:
- Commit SHA pushed
- Files added/changed
- Pages URL (the eventual `https://<owner>.github.io/<repo>/`)
- Whether Pages source needs to be set manually
- Whether "About" was updated automatically or needs manual entry
- Any secret-scan warnings the user should know about

## Rules

- Never use `git push --force`, `--no-verify`, or `git add -A` / `git add .` in this workflow.
- Never commit `.env`, key files, or anything containing live credentials, even if the user insists — ask them to use repository Secrets (`Settings → Secrets and variables → Actions`) and reference via `${{ secrets.NAME }}` in workflows.
- If a pre-commit hook fails, fix the underlying issue and make a NEW commit. Do not `--amend` or bypass hooks.
- If the working tree is dirty with changes unrelated to this push, ask the user before staging them.
