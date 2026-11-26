# Problem PDF Viewer

## Table of Contents

- [What This Achieves](#what-this-achieves)
- [Modes & Features](#modes--features)
- [Run Locally](#run-locally)
- [API Endpoints](#api-endpoints)

## What This Achieves

- **Instant visual review:** Displays any `.pbm` problem's compiled PDF statement alongside its LaTeX, solution, sample tests, and `handler.yml`.
- **Transparent folder scope:** Shows the opened root folder at the top of the sidebar.
- **Open any `.pbm` from PC:** Includes a browser folder picker (`Open .pbm from PC`) that works client-side on both GitHub Pages and localhost, plus server directory switching locally.
- **Zero heavy stack:** Built with `Bun.serve` + plain HTML/CSS/JS (no database, auth, or framework).

## Modes & Features

| Mode | Where It Runs | Capabilities |
|---|---|---|
| **Live Server (`server.ts`)** | Localhost (`PORT=3456`) | Scans folder, auto-builds PDFs via `jtk make pdf` (+ Chrome fallback), **Rebuild PDF** / **`jtk make`** / **`jtk lint`** buttons, `Switch Server Folder` |
| **Static Pages (`export-static.ts`)** | GitHub Pages (`pedit-demo`) | Prebuilt `manifest.json` + PDFs + sanitized `files.json` + client-side **Open `.pbm` from PC** |

## Run Locally

```bash
cd viewer
bun run server.ts
# Open http://localhost:3456
# Optional custom folder:
PEDIT_PROBLEMS_DIR=/path/to/problems bun run server.ts
```

## API Endpoints

- `GET /api/problems`: Returns opened `root` / `sourceLabel` and `.pbm` list (excludes `private-problems/` unless `?private=1`).
- `POST /api/open-dir`: Switches the scanned local directory on the live server.
- `GET /api/problems/:id/pdf?lang=<lang>&rebuild=0`: Serves `problem.<lang>.pdf` (builds if missing or `rebuild=1`).
- `GET /api/problems/:id/files`: Returns inspectable text files (never exposes `problem.yml`).
- `POST /api/problems/:id/action`: Runs `make`, `pdf`, `lint`, or `verify` via unmodified `jtk`.
