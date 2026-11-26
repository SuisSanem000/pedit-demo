# `pedit` CLI

## Table of Contents

- [What This Achieves](#what-this-achieves)
- [Install & Run](#install--run)
- [Commands](#commands)
- [How `pedit` Differs from `jtk`](#how-pedit-differs-from-jtk)
- [Architecture](#architecture)

## What This Achieves

- **Keeps `jtk` unchanged:** Wraps `jtk` (`4.4.50`) as the deterministic build, test, and lint engine with `--json` output.
- **Selectable low-cost AI:** Uses `@openrouter/sdk` (default `openai/gpt-4o-mini`; supports `google/gemini-2.5-flash-lite` and `deepseek/deepseek-chat-v3-0324`).
- **Hidden master prompts:** Loads `spec/constitucio.md`, `spec/problema-jutge.md`, and `skills/problemes-jutge/SKILL.md` automatically so users only type short ideas or edit requests.
- **Closed-loop safety:** Blocks LLM writes to `*.cor` and `problem.yml`; runs `generate.ts`, `jtk make`, PDF build, and `jtk lint` with up to 2 automatic retries.

## Install & Run

```bash
cd pedit
bun install
export OPENROUTER_API_KEY="your-key"   # or ANTHROPIC_AUTH_TOKEN when routed via OpenRouter
bun run src/index.ts --help
```

## Commands

| Group | Command | Purpose |
|---|---|---|
| **Inspect** | `pedit list [--dir ..] [--json]` | Find `.pbm` folders (excludes `private-problems/` by default) |
| **Inspect** | `pedit status <pbm>` / `pedit show <pbm> [file]` | Inspect languages, PDFs, tests, or file contents |
| **Build (`jtk`)** | `pedit build` / `pdf` / `lint` / `verify` / `clean <pbm>` | Run unmodified `jtk` commands and return JSON |
| **Direct Edit** | `pedit meta <pbm> --lang en --title "..."` | Update `problem.<lang>.yml` without AI |
| **Direct Edit** | `pedit handler <pbm> --compilers G++17` | Update `handler.yml` without AI |
| **AI Authoring** | `pedit create <dir.pbm> "<idea>"` | Generate `.pbm` problem + verify in closed loop |
| **AI Editing** | `pedit edit <pbm> ["<req>"] [--preset <name>]` | Edit `.pbm` via OpenRouter + preset shortcuts |
| **Interactive** | `pedit chat <pbm>` | Multi-turn terminal editing loop |
| **Discovery** | `pedit models` / `pedit presets` | List low-cost models and edit presets |

## How `pedit` Differs from `jtk`

| Dimension | `jtk` (Existing) | `pedit` (New CLI) |
|---|---|---|
| **Primary Role** | Deterministic build, test, PDF, lint, upload | AI-assisted problem authoring & editing |
| **LLM Provider** | Fixed Jutge API (`openai/gpt-6-luna`) | OpenRouter SDK (default `openai/gpt-4o-mini`; Jutge API later) |
| **Workflow** | One-shot `jtk generate` | Conversational `edit` / `chat` + edit presets |
| **Prompts / Rules** | Hardcoded in CLI | Hidden master prompts loaded from `spec/` & `skills/` |
| **Verification** | Manual | Closed-loop (`jtk` runs automatically with retries) |

## Architecture

- `src/index.ts`: CLI flag parser and command router.
- `src/jtk.ts`: Subprocess runner with timeouts for `jtk` and `bun run generate.ts`.
- `src/pbm.ts`: `.pbm` scanner, YAML editors, and write guard (`assertSafeRelativePath`).
- `src/pdf.ts`: `jtk make pdf` runner + headless Chrome HTML-to-PDF fallback.
- `src/ai.ts`: `@openrouter/sdk` client, preset prompts, and closed-loop retry runner.
- `tests/pedit.test.ts`: Unit tests (`bun test`).
