# Deploy Guide — Static Site on Render

This is a **plain static HTML/CSS/JS site**. No build step, no Node, no npm — just files served by nginx in a small Docker image. You can deploy in **under 5 minutes**.

## Prerequisites

- A [Render](https://render.com) account (free to sign up)
- This repository pushed to GitHub, GitLab, or Bitbucket
- (Optional, for CI/CD) Docker installed locally to test before deploying

---

## Option A: Deploy via Render Blueprint (fastest — recommended)

1. Push this repo (including `render.yaml`) to GitHub.
2. Go to the Render Dashboard → **New** → **Blueprint**.
3. Connect your repository. Render will detect `render.yaml` automatically.
4. Click **Apply** — Render builds the Docker image and deploys it.
5. Your site is live at `https://<your-service-name>.onrender.com`.

That's it — 3 clicks, no CLI needed.

---

## Option B: Deploy via Render Dashboard manually

1. Go to Render Dashboard → **New** → **Web Service**.
2. Connect your repo.
3. Set:
   - **Runtime**: `Docker`
   - **Dockerfile Path**: `./Dockerfile`
   - **Health Check Path**: `/healthz`
   - **Plan**: `Starter` (or Free, if available)
4. Click **Create Web Service**.

---

## Option C: Local test before deploying (3 commands)

```bash
# 1. Build the image
docker build -t static-site:local .

# 2. Run it locally on port 8080
docker run -p 8080:8080 static-site:local

# 3. Verify it's healthy
curl http://localhost:8080/healthz
```

Open `http://localhost:8080` in your browser to preview the site exactly as it will run in production.

Or with docker-compose:

```bash
docker compose up --build
```

---

## Enabling automatic deploys via GitHub Actions (optional)

1. In Render, open your Web Service → **Settings** → **Deploy Hook** → copy the URL.
2. In GitHub, go to your repo → **Settings** → **Secrets and variables** → **Actions**.
3. Add a new secret named `RENDER_DEPLOY_HOOK_URL` with the copied URL.
4. Every push to `main` will now: lint HTML → build Docker image → run smoke test → scan for vulnerabilities → trigger Render deploy.

---

## Troubleshooting

| Problem | Fix |
|---|---|
| Health check fails | Confirm container listens on port `8080` and `/healthz` returns `200` |
| 404 on a page | Ensure the `.html` file is copied in the `Dockerfile` `COPY` list |
| Assets not loading | Confirm files exist under `assets/` and are copied in the Dockerfile |
| Render build fails | Check the Render build logs — usually a missing file in `COPY` |

---

## File Reference

- `Dockerfile` — builds the nginx image serving the static files
- `nginx.conf` — nginx server block with security headers, gzip, health check
- `render.yaml` — Render Blueprint config (Docker runtime)
- `docker-compose.yml` — local dev/preview only
- `.github/workflows/deploy.yml` — CI pipeline (lint → build → scan → deploy)