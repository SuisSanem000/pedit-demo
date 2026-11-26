# `pedit` CLI & Problem PDF Viewer

## Table of Contents

- [What This Achieves](#what-this-achieves)
- [1. `pedit` CLI](#1-pedit-cli)
- [2. Problem PDF Viewer](#2-problem-pdf-viewer)
- [Architecture](#architecture)

## What This Achieves

- **Unified toolset:** Combines the AI-assisted `pedit` CLI and the Problem PDF Viewer in one folder (`pedit/`), wrapping unmodified `jtk` (`4.4.50`).
- **Low-cost OpenRouter AI:** Uses `@openrouter/sdk` (default `openai/gpt-4o-mini`), hidden `spec/` & `skills/` prompts, and closed-loop `jtk` retries.
- **Visual PDF review:** Lets teachers inspect compiled `.pbm` PDFs, LaTeX, solutions, and sample tests locally (`pedit viewer`) or on GitHub Pages (including **Open `.pbm` from PC**).

## 1. `pedit` CLI

```bash
cd pedit && bun install
export OPENROUTER_API_KEY="your-key"   # or ANTHROPIC_AUTH_TOKEN when routed via OpenRouter

bun run src/index.ts list --dir .. --json
bun run src/index.ts status <problem.pbm> --json
bun run src/index.ts build|pdf|lint|verify|clean <problem.pbm>
bun run src/index.ts create <dir.pbm> "<idea>"
bun run src/index.ts edit <dir.pbm> ["<request>"] [--preset add-edge-tests]
bun run src/index.ts chat <dir.pbm>
```

| Dimension | `jtk` (Unchanged) | `pedit` (Unified Tool) |
|---|---|---|
| **Role** | Deterministic build, test, PDF, lint, upload | AI authoring, editing & PDF viewer |
| **Model** | Fixed Jutge API (`openai/gpt-6-luna`) | OpenRouter SDK (default `openai/gpt-4o-mini`) |
| **Prompts** | Hardcoded | Hidden `spec/` & `skills/` |
| **Loop** | One-shot | Closed-loop `jtk` retries (blocks `*.cor`/`problem.yml` writes) |

## 2. Problem PDF Viewer

```bash
bun run src/index.ts viewer --dir .. --port 3456
# Or export static GitHub Pages bundle:
bun run export ./dist-pages
```

- **Live mode (`src/viewer-server.ts`):** Scans folder, auto-builds PDFs (`jtk make pdf` + Chrome fallback), and exposes **Rebuild PDF**, **`jtk make`**, **`jtk lint`**, and `Switch Server Folder`.
- **Static mode (`src/export-static.ts`):** Exports `index.html`, `manifest.json`, PDFs, and sanitized `files.json` + client-side **Open `.pbm` from PC**.

## Architecture

- `src/index.ts` — CLI router (`list`, `status`, `show`, `build`, `pdf`, `lint`, `verify`, `clean`, `meta`, `handler`, `models`, `presets`, `create`, `edit`, `chat`, `viewer`).
- `src/jtk.ts` — Subprocess wrapper for `jtk` and `bun run generate.ts`.
- `src/pbm.ts` — `.pbm` scanner, YAML updaters, and write guard (`assertSafeRelativePath`).
- `src/pdf.ts` — `jtk make pdf` runner + headless Chrome HTML-to-PDF fallback.
- `src/ai.ts` — `@openrouter/sdk` client and closed-loop retry engine.
- `src/viewer-server.ts` & `src/export-static.ts` — Local server and static Pages exporter.
- `public/index.html` — Unified single-page UI (CLI at top, PDF Viewer below).
- `tests/pedit.test.ts` — Unit tests (`bun test`).
