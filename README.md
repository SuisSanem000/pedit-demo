# `pedit` CLI & Problem PDF Viewer (Public Demo)

**Live GitHub Pages:** [Demo Hub](https://suissanem000.github.io/pedit-demo/) • [pedit CLI](https://suissanem000.github.io/pedit-demo/cli/) • [PDF Problem Viewer](https://suissanem000.github.io/pedit-demo/pdf-viewer/)

> Public mirror containing only self-authored tooling (`pedit/`, `viewer/`) and synthetic demo problems (`demo-problems/`). No upstream professor or private exam materials are included.

## Table of Contents

- [Quick Start](#quick-start)

## Quick Start

```bash
# 1. pedit CLI (wraps unmodified jtk)
cd pedit && bun install
bun run src/index.ts --help

# 2. Local Problem PDF Viewer (Bun.serve)
cd ../viewer && PEDIT_PROBLEMS_DIR=../demo-problems bun run server.ts
```
