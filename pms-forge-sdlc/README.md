## About this project

This repository is published as an open, public reference. Anyone can read the
code, the issue history, and the SDLC process itself (see "What it is" above) to
see how AI-assisted delivery with human approval gates works end to end.

### Where this comes from

This repository is the technical companion to the project showcased at
**[Primal Motion Studio — PRJ05](https://primalmotionstudio.wixstudio.com/home/prj05)**.
If you arrived here from that page, you're in the right place: this is the actual
source code, history, and process behind it.

### Get in touch

If you'd like to know more, discuss the project, or ask about access to anything
not covered here, feel free to reach out at https://www.linkedin.com/in/juanceballos/ .

------------------------------

# pms-forge-sdlc

GitHub-native SDLC orchestration — human-supervised, AI-assisted software delivery.

## What it is

A single Python script (`src/bootstrap.py`) installs a lightweight SDLC framework into
any GitHub repository. Once installed, GitHub Issues drive a lifecycle where AI agents
draft proposals and task breakdowns, and humans approve every transition.

```
idea → triage → planned → ready → implemented → reviewed → done
```

AI assists at `state:triage` (writes `proposal.md`) and `state:planned` (writes `tasks.md`).
Humans approve at every gate. Nothing merges automatically.

## Quick start

### 1. Get the bootstrap script

```bash
git clone https://github.com/juanceballos/pms-forge-sdlc
cd pms-forge-sdlc
```

### 2. Run bootstrap on your target repo

```bash
# Framework only — any project
python src/bootstrap.py --repo your-org/your-repo --token ghp_xxx

# With an application scaffold (Dev Assist included by default)
python src/bootstrap.py --repo your-org/your-repo --token ghp_xxx --app-dir my-app
```

Replace `your-org/your-repo` with your GitHub username/org and repo name.
Get a token at **GitHub → Settings → Developer settings → Personal access tokens**.
Required scopes: `repo` (full), `workflow`.

### 3. Add required secrets and enable Actions write access

**Settings → Secrets and variables → Actions → New repository secret**

| Secret | Value |
|--------|-------|
| `ANTHROPIC_API_KEY` | Your key from console.anthropic.com |
| `RENDER_DEPLOY_HOOK` | Webhook URL from Render service settings (see below) |

**Manual one-time settings — required for Dev Assist (Claude Code agent):**

1. Install the Claude Code GitHub App on your repo: **https://github.com/apps/claude**
2. GitHub repo → **Settings → Actions → General** → check **"Allow GitHub Actions to create and approve pull requests"**

> **If the deploy step returns a 404:** the `RENDER_DEPLOY_HOOK` URL is stale. Render dashboard → service **Settings → Deploy Hook** → regenerate the URL, then update the secret in GitHub.

### 4. Set up deployment to Render (one-time, ~10 min)

The bootstrap generates a Docker-based deploy pipeline: `state:review` triggers a build,
pushes to GitHub Container Registry (GHCR), and Render redeploys automatically.

**Step 1 — Create a Render account and web service (5 min)**

1. Go to [render.com](https://render.com) → sign up (free, no credit card required)
2. New → **Web Service** → **Deploy an existing image**
3. Image URL: `ghcr.io/<your-github-username>/<repo-name>:latest`
4. Name your service, pick the region closest to you
5. Instance type: **Free**
6. Click **Create Web Service**

Render gives you a stable public URL: `https://your-service-name.onrender.com`

> **Free tier note:** services sleep after 15 min of inactivity. First request after idle
> takes ~30 s to wake. Fine for team testing; revisit Fly.io (~$5/month) if you need
> always-on behaviour.

**Step 2 — Get the Deploy Hook URL (1 min)**

Render dashboard → your service → **Settings** → scroll to **Deploy Hook** → copy the URL
(looks like `https://api.render.com/deploy/srv-xxx?key=yyy`)

Add it to GitHub: **Settings → Secrets → Actions → New secret → `RENDER_DEPLOY_HOOK`**

**Step 3 — Make the GHCR package public (2 min, after first push)**

The first push to the repo (or `state:review` label) triggers a Docker build and pushes
the image to GHCR. After that first push:

GitHub → your profile → **Packages** → find the image → **Settings** → **Change visibility → Public**

Render's free tier pulls images without auth, so the package must be public.

**How the full flow works after setup**

```
push to any branch (including merges to main)
  → docker-build.yml: builds the Docker image and pushes it to GHCR
      - main           → ghcr.io/<owner>/<repo>:latest
      - other branches → ghcr.io/<owner>/<repo>:preview
      - always also    → ghcr.io/<owner>/<repo>:sha-<short-sha>
  → deploy-render.yml: POSTs to RENDER_DEPLOY_HOOK
  → Render pulls the latest image and redeploys
  → https://your-service.onrender.com is live (~3-5 min total)

state:review label applied
  → docker-build.yml also builds/pushes an image on demand (same tagging rules)
```

**Local development with Docker**

```bash
# Run the app locally via Docker (Vite dev server with hot reload)
cd demo-app/docker
docker compose up

# App available at http://localhost:3000
```

**Switching deployment targets later**

To move from Render to Fly.io or another platform, update `sdlc.yml`:
```yaml
deploy:
  platform: fly   # was: render
```
Then replace the `deploy-render.yml` workflow with a platform-specific equivalent.
The `docker-build.yml` workflow is platform-agnostic and stays unchanged.

### 5. Create your first story

**Fully automated** — apply both labels at once:
1. Open a GitHub Issue using the **Story** issue template
2. Apply `state:triage` + `sdlc:auto`
3. PO → PE → Dev run automatically; one draft PR accumulates all artifacts
4. Review the PR, merge, apply `state:done`

**Manual gates** — apply `sdlc:auto` only when you're ready to hand off each step:
1. Apply `state:triage` → PO generates `proposal.md`, review on branch
2. Apply `state:planned` → PE generates `tasks.md`, review on branch
3. Apply `state:ready` → Dev generates implementation (or use Claude Code)
4. Review PR, merge, apply `state:done`

## This repo as a demo

`demo-app/` shows the SDLC applied to a real React dashboard project.
`demo-app/pms-sdlc/` holds the framework config, templates, and story history.

See [`demo-app/pms-sdlc/README.md`](demo-app/pms-sdlc/README.md) for full docs,
workflow table, token guide, and repo structure reference.


# Get in touch
If you'd like to know more, discuss the project, or ask about access to anything not covered here, feel free to reach out at https://www.linkedin.com/in/juanceballos/ .
