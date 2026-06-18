# Automation: Lucio AI Briefcase

**Automatically downloads legal & regulatory documents from 90+ government websites, for any date range.**

A self-contained Python tool that scrapes **93 regulatory document sources** across **15+ Indian and international regulators** in real time. Pick a date range, and it pulls every matching filing (Red Herring Prospectuses, merger orders, circulars, court judgments, consultation papers) and saves the PDFs into neatly organised folders on your machine. Run it as a **live web dashboard** or as a **one-file command-line tool**.

![Python](https://img.shields.io/badge/python-3.8%2B-blue?logo=python&logoColor=white)
![Sources](https://img.shields.io/badge/sources-93-green)
![Dependencies](https://img.shields.io/badge/runtime-stdlib--only-brightgreen)
![Platform](https://img.shields.io/badge/runs-local%20%7C%20cloud-informational)
![License](https://img.shields.io/badge/license-MIT-blue)

---

## Table of contents

- [What it does](#what-it-does)
- [Why it's useful](#why-its-useful)
- [Requirements](#requirements)
- [Quick start](#quick-start)
- [The two ways to run it](#the-two-ways-to-run-it)
- [Choosing any date range](#choosing-any-date-range)
- [Where your files are saved](#where-your-files-are-saved)
- [Covered sources (all 93)](#covered-sources-all-93)
- [Repository layout](#repository-layout)
- [How it works (architecture)](#how-it-works-architecture)
- [API reference](#api-reference)
- [Helper scripts](#helper-scripts)
- [Configuration](#configuration)
- [Deploying online (optional)](#deploying-online-optional)
- [Troubleshooting](#troubleshooting)
- [Legal & responsible use](#legal--responsible-use)
- [Author & license](#author--license)

---

## What it does

On startup the tool reaches out to dozens of regulatory bodies at once and collects every document filed within the date range you choose. You can then:

- **Browse** documents grouped by regulator (SEBI, BSE, CCI, RBI, IRDAI, RERA, TRAI, GST, EU, UK, UAE, …).
- **Filter** by date range and **search** by company name or document title.
- **Download** the source PDFs straight to organised local folders, one per source type.
- **Bulk download** an entire category, or export everything as a single **ZIP** or an **Excel** workbook.
- **Audit** accuracy: run a verification pass that HEAD-checks every cached PDF URL and reports broken links.
- **Export** the audit as CSV or JSON.

Everything runs **on your own machine**. No account, no API keys, and no third-party service is required to use it locally.

## Why it's useful

Regulatory filings are scattered across dozens of government portals, each with its own search form, pagination quirks, and (often) a bot-blocking firewall. Collecting a month's worth of filings by hand means visiting 90+ pages, fighting each search UI, and downloading PDFs one at a time.

This tool does all of that in one pass:

- **One date range, every regulator:** set "From" and "To" once; every source is queried for that window.
- **Resilient fetching:** sends full browser-like headers, uses a `curl` fallback for HTTP/2-only domains, and fetches SEBI sequentially with deliberate gaps to stay under its Web Application Firewall (WAF) radar.
- **Organised output:** PDFs land in `~/Desktop/Repositories/<SourceType>/`, ready to hand off or archive.
- **Zero install friction:** pure Python standard library at runtime (one optional package only, for Excel export).

---

## Requirements

- **Python 3.8 or newer** (ships with macOS and most Linux distributions).
- **`curl`** on your `PATH`, used as a fallback for a few HTTP/2-only government domains (preinstalled on macOS and Linux).
- That's it. The tool uses **only the Python standard library at runtime**.
- **Optional:** [`openpyxl`](https://pypi.org/project/openpyxl/), required *only* for the "Export to Excel" button. Install with `pip install -r requirements.txt`. Everything else works without it.

> **No `pip install` is needed to browse, search, download PDFs, or run the audit.** The single optional dependency exists purely for the `.xlsx` export feature.

---

## Quick start

```bash
# 1. Get the code
git clone https://github.com/mohitlucio/Automation.git
cd Automation

# 2. (Optional) enable the Excel-export feature
pip install -r requirements.txt

# 3a. Launch the web dashboard …
python3 backend/server.py
#     → opens http://localhost:8765 in your browser

# 3b. … or run the command-line version
python3 briefcase.py
```

**macOS users** can skip the terminal entirely and double-click:

| File | What it launches |
|---|---|
| `Start_Briefcase.command` | The **web dashboard** (opens `http://localhost:8765` automatically) |
| `Briefcase.command` | The **command-line** runner (`briefcase.py`) |

> The first time you double-click a `.command` file, macOS may ask you to confirm it (right-click → *Open*). Close the Terminal window to stop the server.

---

## The two ways to run it

This project ships **two front-ends over the same scraping engine** (`backend/server.py`). Pick whichever fits the moment.

### 1. Web dashboard (`python3 backend/server.py`)

A self-contained single-page dashboard served at **`http://localhost:8765`**.

- Category tabs for every regulator, each with a live document count.
- A global progress bar plus per-source status dots (scraping → ready / error).
- Date-range and search controls at the top.
- A full-screen **Downloads** manager (in-progress / completed / failed) with one-click bulk and ZIP download.
- An **Accuracy Audit** panel that verifies every cached PDF link and exports CSV/JSON.

The dashboard UI is a single file: `frontend/index.html` (HTML + CSS + JS, no build step, no framework).

### 2. Command-line runner (`python3 briefcase.py`)

A one-file interactive script for when you just want the PDFs with no browser involved.

1. It asks for a **From** and **To** date (defaults to the current month).
2. It scrapes all 93 sources, printing a live colour-coded status for each.
3. It lists everything it found, then asks before downloading every PDF to `~/Desktop/Repositories/`.

`briefcase.py` imports the same scraper module the server uses, so both paths produce identical results.

---

## Choosing any date range

Both front-ends let you scrape **any time period**, not just the current month:

- **Dashboard:** use the *From* / *To* date pickers and refresh; internally this calls `POST /api/set_range` (a single month can also be selected via `POST /api/set_month`).
- **Command line:** type the dates when `briefcase.py` prompts you (press Enter to accept the current-month default).
- **Headless / automation:** `python3 scrape_to_json.py --month YYYY-MM` scrapes a specific month and writes `docs/data.json`.

Each source receives the date window and returns only the filings published within it.

---

## Where your files are saved

By default, downloads are written to:

```
~/Desktop/Repositories/<SourceType>/<document>.pdf
```

For example: `~/Desktop/Repositories/RHP/`, `~/Desktop/Repositories/CCI_Form1/`, `~/Desktop/Repositories/EU_Mergers/`. Each of the 93 sources gets its own sub-folder, so nothing collides.

You can change the destination with the `DOWNLOAD_DIR` environment variable (see [Configuration](#configuration)).

---

## Covered sources (all 93)

| Category | Regulators / Sources |
|---|---|
| **SEBI** | RHP, DRHP, Rights LoF, InvIT (6 types), REIT (Final/Draft), Informal Guidance, Consultation Papers, Circulars, Final Offer, LODR, ICDR, Takeover, AIF |
| **BSE** | QIP Placement Documents, Draft/Preliminary Placement |
| **CCI** | Form I / II / III, Gun Jumping, Approved-with-Modification, Antitrust (S.26(1)/(2)/(6)/(7), S.27, S.33, Other), Green Channel |
| **RBI** | Master Directions, Master Circulars, 9 entity types (Commercial Banks, SFBs, Payment Banks, …), FEMA Directions / Circulars / Notifications |
| **IRDAI** | Circulars, Gazette-notified Regulations |
| **India INX** | Circulars & Notices, Issuer Documents |
| **RERA** | Telangana (4 types), Tamil Nadu, Maharashtra, Karnataka (REAT + RERA), Haryana REAT, Delhi REAT, DTCP Karnataka |
| **TRAI** | Directions, Regulations, Recommendations, Consultation Papers |
| **GST** | CGST Circulars |
| **IBBI / NCLT** | Resolution & Admission Orders |
| **EU Competition** | Mergers (Reg. 139/2004), Antitrust & Cartels, DMA, Foreign Subsidies |
| **EPO** | Board of Appeal Decisions |
| **GDPR (EDPB)** | Guidelines, Binding Decisions, Opinions |
| **UAE** | ADGM Court Judgments, DIFC Court of Appeal, MOHRE Laws & Resolutions |
| **UK Tribunals** | Employment (England & Scotland), Admin Appeals, CAT, CMA (Mergers & Non-merger), EAT, UTIAC, Land Chamber, Tax Chancery, Tax FTT |

> The complete, authoritative list of source keys lives in the `SOURCES` dictionary at the top of [`backend/server.py`](backend/server.py).

---

## Repository layout

```
Automation/
├── README.md                  ← this file
├── LICENSE                    ← MIT
├── requirements.txt           ← optional dependency (openpyxl, for Excel export only)
├── .env.example               ← environment-variable reference
├── .gitignore / .dockerignore
│
├── briefcase.py               ← One-file CLI runner (no server, no browser)
├── Briefcase.command          ← macOS double-click → runs briefcase.py
├── Start_Briefcase.command    ← macOS double-click → runs the web dashboard
│
├── backend/
│   └── server.py              ← HTTP server + all 93 scrapers (standard library only)
├── frontend/
│   └── index.html             ← Single-file dashboard (HTML + CSS + JS, no build step)
│
├── scrape_to_json.py          ← Headless scraper → writes docs/data.json (used by CI / Pages)
├── control.py                 ← Remote-control CLI for the hosted site (dispatches gh workflows)
├── sebi_browser_scrape.py     ← Chrome-assisted SEBI fetch (WAF fallback, macOS)
│
├── docs/                      ← Static GitHub Pages demo + a sample data snapshot
│   ├── index.html
│   ├── data.json
│   ├── scrape_log.json
│   └── commands.json
│
├── deployment/                ← Everything about hosting it online (start at deployment/README.md)
│   ├── README.md
│   ├── ARCHITECTURE.md
│   ├── DEPLOYMENT.md
│   ├── DEPLOY_QUICK.md
│   ├── DEPLOY_CHOOSE.md
│   ├── RAILWAY_DEPLOYMENT.md
│   ├── SETUP_COMPLETE.md
│   └── scripts/               ← deploy-cloudrun.sh, deploy-flyio.sh, deploy-complete.sh, …
│
├── ops/                       ← Caddyfile + VPS bootstrap (used by the deploy-vps workflow)
│
├── Dockerfile                 ← container image
├── docker-compose.prod.yml    ← app + Caddy reverse proxy
├── Procfile                   ← Heroku / Render process definition
├── render.yaml / render.json  ← Render.com config
├── railway.json               ← Railway config
├── fly.toml                   ← Fly.io config
│
└── .github/workflows/         ← deploy.yml, deploy-vps.yml, scrape.yml (manual triggers, see note below)
```

---

## How it works (architecture)

**Backend (`backend/server.py`, standard library only):**

- **Server:** a `ThreadingHTTPServer` listening on `localhost:8765` (locally) or `0.0.0.0:$PORT` (in the cloud).
- **Concurrency model:** SEBI sources are scraped **sequentially with deliberate gaps** to avoid tripping SEBI's WAF; all non-SEBI sources run **in parallel** through a `ThreadPoolExecutor`.
- **Browser-like requests:** outgoing requests send a full modern-Chrome header set (`Sec-Fetch-*`, `Accept-Language`, etc.). Several government WAFs return `403` to "thin" requests; looking like a real browser is what gets through.
- **`curl` fallback:** a few domains only speak HTTP/2; for those the server shells out to `curl`.
- **Watchdog:** any scraper stuck for too long is reset automatically, so one slow source can't stall the whole run.
- **Downloads:** a global semaphore caps concurrent downloads so the tool stays a polite client.
- **Storage:** PDFs are saved under `~/Desktop/Repositories/<SourceType>/` (configurable via `DOWNLOAD_DIR`).

**Frontend (`frontend/index.html`):**

- A single self-contained page: no framework, no bundler, no build step.
- Polls the backend for per-source status and renders live progress dots, category counts, the downloads manager, and the audit panel.

A deployment-oriented architecture diagram (local vs. cloud, CI/CD flow) lives in [`deployment/ARCHITECTURE.md`](deployment/ARCHITECTURE.md).

---

## API reference

The backend exposes a small JSON API. The most useful endpoints:

| Endpoint | Method | Description |
|---|---|---|
| `/` | GET | Serves the dashboard (`frontend/index.html`) |
| `/health` | GET | Health check (used by cloud platforms) |
| `/api/status` | GET | Per-source cache state and progress |
| `/api/active_month` | GET | Currently selected month and date range |
| `/api/documents?type=RHP` | GET | Documents for a single source |
| `/api/unified?from=&to=&search=` | GET | All sources merged, date-filtered, searchable |
| `/api/search?q=` | GET | Search across cached documents |
| `/api/set_month` | POST | Set the active scraping month |
| `/api/set_range` | POST | Set an arbitrary **From / To** date range |
| `/api/refresh` | POST | Re-scrape (optionally specific sources) |
| `/api/download` | POST | Download a single PDF |
| `/api/download_all` | POST | Download every document for a source/category |
| `/api/download_progress` | GET | Live download progress |
| `/api/download_zip` | GET | Download a category as a single ZIP |
| `/api/export_excel` | GET | Export results as an `.xlsx` workbook (needs `openpyxl`) |
| `/api/audit` | GET | HEAD-verify every cached PDF URL |
| `/api/retry_errors` | POST | Retry sources that errored |
| `/api/stop` | POST | Stop in-flight scraping |

Additional internal endpoints (`/api/get_pdf_url`, `/api/manual_download`, `/api/browser_inject`, `/api/reset`, …) support the dashboard's advanced features.

---

## Helper scripts

These live at the repository root because they import `backend/` and write to `docs/` using paths relative to themselves, so keep them where they are.

| Script | Purpose |
|---|---|
| [`scrape_to_json.py`](scrape_to_json.py) | Runs the scrapers **headlessly** and writes the results to `docs/data.json`, powering the static GitHub Pages demo. Supports `--sources RHP,DRHP` and `--month YYYY-MM`. |
| [`control.py`](control.py) | A command channel for the **hosted** site. Uses the GitHub CLI (`gh`) to trigger the scrape workflow, post a notice banner, or change the active month remotely. |
| [`sebi_browser_scrape.py`](sebi_browser_scrape.py) | A macOS-only fallback that drives **Google Chrome** via AppleScript to fetch SEBI pages a real browser can reach when the plain HTTP path is blocked. |

---

## Configuration

Configuration is entirely through environment variables (see [`.env.example`](.env.example)). Sensible defaults mean you normally don't need to set anything for local use.

| Variable | Default (local) | Description |
|---|---|---|
| `CLOUD` | unset (`0`) | Set to `1` for cloud mode (binds `0.0.0.0`, doesn't auto-open the browser, uses the cloud storage path). |
| `PORT` | `8765` | Port the server listens on (cloud platforms usually inject `10000`). |
| `DOWNLOAD_DIR` | `~/Desktop/Repositories` | Where downloaded PDFs are written. In cloud mode this defaults to a writable container path. |
| `RENDER` | unset | Auto-detected when running on Render.com; enables cloud behaviour. |

---

## Deploying online (optional)

The tool is **local-first**; that's the primary experience. If you want to share a live URL with others, the [`deployment/`](deployment/) folder has guides and ready-made configs for **Render, Railway, Fly.io, Google Cloud Run, Docker, and a self-hosted VPS (with Caddy + HTTPS)**.

Start with **[`deployment/README.md`](deployment/README.md)**.

### A note on the GitHub Actions workflows

The three workflows under `.github/workflows/` (`deploy.yml`, `deploy-vps.yml`, `scrape.yml`) ship with their **automatic triggers disabled** (`workflow_dispatch` / manual only). This keeps a freshly-published repository green: they won't fail for lack of deployment secrets, and the scraper won't auto-commit data into your history.

Each file documents, in a comment at the top, exactly which trigger to uncomment (and which secrets to set) to turn on continuous deployment or the scheduled 6-hourly scrape.

### Hosted demo (GitHub Pages)

A live, hosted demo is published from the `docs/` folder via GitHub Pages: **https://mohitlucio.github.io/Automation/**. It shows the dashboard with a saved data snapshot, so anyone can open it in a browser with nothing to install. (Served from the `main` branch, `/docs` folder.)

---

## Troubleshooting

| Symptom | Fix |
|---|---|
| **A source shows an error dot** | Government portals go down or rate-limit intermittently. Use the dashboard's *Retry errors* button (or `POST /api/retry_errors`); it usually clears on a second pass. |
| **SEBI sources are slow** | This is intentional: SEBI is scraped sequentially with gaps to avoid its WAF. Let it finish. |
| **"Export to Excel" does nothing** | Install the optional dependency: `pip install -r requirements.txt`. |
| **Port 8765 already in use** | Set a different port: `PORT=8800 python3 backend/server.py`. |
| **A `.command` file won't open on macOS** | Right-click the file → *Open* the first time to clear Gatekeeper. |
| **`curl: command not found`** | Install curl (`brew install curl` on macOS / `apt install curl` on Linux). A few HTTP/2-only domains need it. |

---

## Legal & responsible use

This tool collects **publicly available** regulatory filings and orders from official government and regulator websites for research, compliance, and archival purposes. It reads public pages only; it does not log in, submit forms on your behalf, or access anything behind authentication.

Please use it responsibly: respect each site's terms of use and `robots.txt`, and keep request volumes reasonable. The built-in rate-limiting (sequential SEBI fetches, capped concurrent downloads) is there to keep the tool a well-behaved client, so please don't remove it.

---

## Author & license

**Built by Mohit Sharma** · [Lucio AI](https://lucioai.com)

Released under the [MIT License](LICENSE).
