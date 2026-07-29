# Deploy Guide (Under 5 Minutes)

This is a **static HTML/CSS/JS site** with no build step. It's containerized with **nginx** (pinned versions) and deployed on **Render** as a Docker web service.

## Prerequisites
- A [Render](https://render.com) account (free to sign up)
- This repo pushed to GitHub/GitLab
- (Optional, for local testing) Docker installed

## Option A: Deploy on Render via Dashboard (fastest, no CLI needed)

1. Go to https://dashboard.render.com → **New** → **Blueprint**
2. Connect your repo — Render will auto-detect `render.yaml`
3. Click **Apply** — Render builds the Dockerfile and deploys automatically

Your site will be live at `https://static-site.onrender.com` (or your chosen name) within ~2-3 minutes.

## Option B: Deploy via Render CLI

```bash
# 1. Install Render CLI
brew install render

# 2. Login
render login

# 3. Deploy using the blueprint in this repo
render blueprint launch
```

## Option C: Test Locally First (recommended before deploying)

```bash
# 1. Build the image
docker build -t static-site:local .

# 2. Run it
docker run -d -p 8080:8080 --name static-site static-site:local

# 3. Verify it's healthy and serving content
curl http://localhost:8080/healthz
open http://localhost:8080
```

Or simply:

```bash
docker compose up --build
```

## CI/CD (Automatic Deploys)

The included `.github/workflows/deploy.yml` will:
1. Validate HTML/CSS/JS files exist and are well-formed
2. Build the Docker image and run a smoke test against `/healthz`
3. On push to `main`, trigger a Render deploy via deploy hook

### One-time setup for auto-deploy:
1. In Render Dashboard → your service → **Settings** → copy the **Deploy Hook URL**
2. In GitHub repo → **Settings** → **Secrets and variables** → **Actions** → add secret:
   - Name: `RENDER_DEPLOY_HOOK_URL`
   - Value: (paste the URL)
3. Push to `main` — deploys happen automatically.

## Troubleshooting
- **Health check failing?** Confirm `/healthz` returns `200 ok` — check `nginx.conf`.
- **404s on routes?** This site serves `.html` files directly (e.g. `/about.html`); pretty URLs like `/about` are also supported via `try_files`.
- **Port issues?** The container listens on `8080` internally — Render maps this automatically via `$PORT` detection for Docker services (Render passes `PORT=8080` by convention; our nginx config is already hardcoded to 8080, which matches Render's default expectation for Docker web services).