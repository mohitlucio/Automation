# Deployment

The Automation tool (Lucio AI Briefcase) is **local-first**; see the [main README](../README.md) to run it on your own machine. This folder is only for when you want to **host it online** and share a live URL.

> **TL;DR:** the fastest path is **Google Cloud Run**. From the repository root:
> ```bash
> bash deployment/scripts/deploy-cloudrun.sh
> ```

---

## Pick a platform

Start with **[DEPLOY_CHOOSE.md](DEPLOY_CHOOSE.md)**, a side-by-side comparison of Cloud Run, Fly.io, and a self-hosted VPS (cost, free tier, timeouts, setup time).

| Guide | What it covers |
|---|---|
| **[DEPLOY_CHOOSE.md](DEPLOY_CHOOSE.md)** | Compare the options and pick one. |
| **[DEPLOY_QUICK.md](DEPLOY_QUICK.md)** | The 5-step fast path (GitHub → Render, auto-deploy on push). |
| **[DEPLOYMENT.md](DEPLOYMENT.md)** | The full, detailed deployment guide. |
| **[RAILWAY_DEPLOYMENT.md](RAILWAY_DEPLOYMENT.md)** | Deploying on Railway specifically. |
| **[ARCHITECTURE.md](ARCHITECTURE.md)** | Local vs. cloud architecture and the CI/CD flow, as diagrams. |
| **[SETUP_COMPLETE.md](SETUP_COMPLETE.md)** | Post-setup checklist and summary. |

## Helper scripts (`scripts/`)

Run these from the **repository root** so they can find the project files:

| Script | Target |
|---|---|
| `scripts/deploy-cloudrun.sh` | Google Cloud Run (one command, recommended) |
| `scripts/deploy-flyio.sh` | Fly.io (always-free tier) |
| `scripts/deploy-complete.sh` | Your own VPS (Docker + Caddy, auto-deploy on push) |
| `scripts/deploy_setup.sh` | Initialise git + first push helper |
| `scripts/guided-deploy.sh` | Interactive wizard: GitHub repo → Render, step by step |

## Platform config files (kept at the repository root)

Each hosting platform auto-detects its own config file, so these intentionally live at the **root**, not in this folder:

| File | Used by |
|---|---|
| `Dockerfile` | Docker / Cloud Run / any container host |
| `docker-compose.prod.yml` | Docker Compose (app + Caddy reverse proxy) |
| `Procfile` | Render / Heroku |
| `render.yaml`, `render.json` | Render.com |
| `railway.json` | Railway |
| `fly.toml` | Fly.io |
| `ops/Caddyfile` | Caddy (HTTPS reverse proxy on a VPS) |

All of them start the app the same way: `python backend/server.py` with `CLOUD=1`.

---

## GitHub Actions (manual by default)

The workflows in [`.github/workflows/`](../.github/workflows/) ship with their **automatic triggers commented out** so a freshly published, secret-less repository stays green:

- **`deploy.yml`**: deploy to Render. Add secrets `RENDER_SERVICE_ID` and `RENDER_API_TOKEN`, then uncomment the `push` trigger.
- **`deploy-vps.yml`**: deploy to a VPS over SSH. Add secrets `VPS_HOST`, `VPS_USER`, `VPS_SSH_KEY`, `APP_DOMAIN`, then uncomment the `push` trigger.
- **`scrape.yml`**: scrape on a schedule and commit `docs/data.json`. Uncomment the `schedule` block to re-enable the 6-hourly refresh once GitHub Pages is set up.

Until you enable a trigger, you can still run any workflow manually from the repository's **Actions** tab (`workflow_dispatch`).

## Cloud behaviour

In cloud mode the app reads these environment variables (the configs above set them for you):

| Variable | Value in cloud | Meaning |
|---|---|---|
| `CLOUD` | `1` | Bind `0.0.0.0`, don't auto-open a browser, use the cloud storage path. |
| `PORT` | `10000` | Port the platform routes traffic to. |
| `DOWNLOAD_DIR` | `/data/Repositories` | Persistent-disk path for downloaded PDFs. |

See [`../.env.example`](../.env.example) for the full reference.
