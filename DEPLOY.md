# Deploy Guide — Static Site on Render (Docker)

This is a **plain static HTML/CSS/JS website** — there is no build step, no npm, no node. It's served by a tiny, pinned `nginx` Docker image.

You can deploy in under 5 minutes using either **Render's dashboard** (easiest) or the **Render CLI**.

---

## Option A — Deploy via Render Dashboard (recommended, ~3 minutes)

1. Push this repository to GitHub (if not already there):
   ```bash
   git add .
   git commit -m "Add production deployment config"
   git push origin main
   ```

2. Go to [render.com](https://dashboard.render.com) → **New** → **Blueprint**.
   Render will detect `render.yaml` automatically and configure the Docker web service for you.

3. Click **Apply** — Render builds the Dockerfile and deploys. Your site will be live at:
   ```
   https://static-site.onrender.com
   ```
   (Render also shows the exact URL in the dashboard once the build finishes.)

That's it — 3 clicks, no CLI needed.

---

## Option B — Deploy via Render CLI

```bash
# 1. Install the Render CLI
brew install render

# 2. Log in
render login

# 3. Deploy using the blueprint in this repo
render blueprint launch
```

---

## Local Test Before Deploying (optional but recommended)

```bash
# Build the image
docker build -t static-site:1.0.0 .

# Run it
docker run --rm -p 8080:8080 static-site:1.0.0

# In another terminal, verify health check
curl http://localhost:8080/healthz
# -> ok
```

Or with docker-compose:

```bash
docker compose up --build
```

Then open http://localhost:8080 in your browser.

---

## Continuous Deployment (already configured)

`.github/workflows/deploy.yml` will automatically:
1. Lint/validate HTML files
2. Build the Docker image and run a smoke test against `/healthz`
3. Trigger a Render deploy hook on every push to `main`

**One-time setup required:** add your Render deploy hook URL as a GitHub secret:

1. In Render dashboard → your service → **Settings** → **Deploy Hook** → copy the URL.
2. In GitHub repo → **Settings** → **Secrets and variables** → **Actions** → **New repository secret**:
   - Name: `RENDER_DEPLOY_HOOK_URL`
   - Value: *(paste the URL)*

After that, every push to `main` auto-deploys.

---

## Troubleshooting

| Problem | Fix |
|---|---|
| Build fails on Render | Check the Dockerfile builds locally first: `docker build -t test .` |
| Health check failing | Confirm `/healthz` returns `ok`: `curl http://localhost:8080/healthz` |
| 404 on routes | This is a static multi-page site — pages must be requested with `.html` (e.g. `/about.html`) |
| Port issues locally | Render sets `$PORT` automatically in production; locally use `-p 8080:8080` |