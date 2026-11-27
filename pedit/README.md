# `pedit` CLI & Problem PDF Viewer

## Table of Contents

- [What This Achieves](#what-this-achieves)
- [1. `pedit` CLI](#1-pedit-cli)
- [2. Problem PDF Viewer](#2-problem-pdf-viewer)
- [How to Verify & Test (For Professors)](#how-to-verify--test-for-professors)
- [Technical Details](#technical-details)

## What This Achieves

- **Unified toolset:** Combines the AI problem-editor CLI and Problem PDF Viewer in `pedit/`, wrapping unmodified `jtk` (`4.4.50`).
- **Low-cost OpenRouter AI:** Uses `@openrouter/sdk` (default `openai/gpt-4o-mini`), hidden `spec/` & `skills/` prompts, and closed-loop `jtk` retries (never writes `*.cor`).
- **Visual PDF review:** Inspect `.pbm` PDFs, LaTeX, solutions, and samples locally (`pedit viewer`) or on GitHub Pages (browser-only **Open `.pbm` from PC** / drag-and-drop).

## 1. `pedit` CLI

### All CLI Commands

| Category | Command | Purpose |
|---|---|---|
| **Inspect** | `pedit list [--dir <path>] [--include-private] [--json]` | Discover `.pbm` folders |
| **Inspect** | `pedit status <problem.pbm> [--json]` | Summarize languages, PDFs, solutions, tests |
| **Inspect** | `pedit show <problem.pbm> [file] [--json]` | Print problem summary or file (blocks `problem.yml`) |
| **Deterministic `jtk`** | `pedit build\|pdf\|lint\|verify\|clean <problem.pbm> [--json]` | Run `generate.ts` + unmodified `jtk` |
| **Deterministic edits** | `pedit meta <problem.pbm> --lang en [--title "..."] [--author "..."]` | Update `problem.<lang>.yml` without AI |
| **Deterministic edits** | `pedit handler <problem.pbm> [--handler std] [--compilers G++17]` | Update `handler.yml` without AI |
| **AI authoring** | `pedit models [--json]` • `pedit presets [--json]` | List cheap OpenRouter models & edit presets |
| **AI authoring** | `pedit create <dir.pbm> "<idea>" [--model <id>] [--json]` | Create + build + lint new `.pbm` |
| **AI authoring** | `pedit edit <dir.pbm> ["<req>"] [--preset <name>] [--json]` | Edit + rebuild + lint existing `.pbm` |
| **AI authoring** | `pedit chat <dir.pbm> [--model <id>]` | Interactive multi-turn editing loop |
| **Web Viewer** | `pedit viewer [--dir <path>] [--port 3456]` | Start local PDF Problem Viewer server |

### Usage & Use Cases

**1. Setup & non-AI commands (work immediately, no API key needed):**

```bash
cd ~/Downloads/pedit && bun install
bun run src/index.ts --help
bun run src/index.ts list --dir .. --json
bun run src/index.ts viewer --dir .. --port 3456
```

**2. AI problem authoring & editing (requires a real `OPENROUTER_API_KEY`):**

```bash
export OPENROUTER_API_KEY="<your-openrouter-key>"
bun run src/index.ts create ./sum-evens.pbm "Read N and N integers, print sum of evens"
bun run src/index.ts edit ./sum-evens.pbm --preset add-edge-tests
bun run src/index.ts edit ./sum-evens.pbm --preset translate-es
```

### Differences from `jtk`

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

- **Live mode (`src/viewer-server.ts`):** Scans folder, auto-builds PDFs (`jtk make pdf` + Chrome fallback), and exposes **Rebuild PDF**, **`jtk make`**, and **`jtk lint`**.
- **Static mode (`src/export-static.ts`):** Exports `index.html`, `manifest.json`, PDFs, and sanitized `files.json` + browser-only **Open `.pbm` from PC** / drag-and-drop (keeps previously opened problems in the sidebar list).

## How to Verify & Test (For Professors)

```bash
cd ~/Downloads/pedit && bun install
bun test
bun run src/index.ts --help
bun run src/index.ts list --dir .. --json
bun run src/index.ts show ../generated-demo/sum-evens.pbm problem.en.tex
bun run src/index.ts build ../generated-demo/sum-evens.pbm --json
bun run src/index.ts lint ../generated-demo/sum-evens.pbm --json
bun run src/index.ts viewer --dir .. --port 3456
```

With `OPENROUTER_API_KEY` set, you can also test live AI closed-loop creation & editing:

```bash
bun run src/index.ts create /tmp/test-sum.pbm "Read two integers a and b and print a+b." --json
bun run src/index.ts edit /tmp/test-sum.pbm --preset add-edge-tests --json
rm -rf /tmp/test-sum.pbm
```

- Note: In `pedit-demo` (public demo mirror), replace `../generated-demo/sum-evens.pbm` with `../demo-problems/sum-evens.pbm`.

## Technical Details

- `src/index.ts` — CLI router (`list`, `status`, `show`, `build`, `pdf`, `lint`, `verify`, `clean`, `meta`, `handler`, `models`, `presets`, `create`, `edit`, `chat`, `viewer`) + path-traversal/`problem.yml` guard on `show`.
- `src/jtk.ts` — Subprocess wrapper for unmodified `jtk` and `bun run generate.ts`.
- `src/pbm.ts` — `.pbm` scanner, YAML updaters, and write guard (`assertSafeRelativePath`).
- `src/pdf.ts` — `jtk make pdf` runner + headless Chrome HTML-to-PDF fallback.
- `src/ai.ts` — `@openrouter/sdk` client, hidden `spec/` & `skills/` prompt loader, and closed-loop retry engine.
- `src/viewer-server.ts` & `src/export-static.ts` — Local Bun server and static Pages exporter.
- `public/index.html` — Unified single-page UI (CLI at top, PDF Viewer below).
- `tests/pedit.test.ts` — Unit tests (`bun test`).
