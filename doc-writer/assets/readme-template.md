# {{ project-name }}

> One-line tagline that describes what this project does and who it's for.

[![CI][badge-ci]][url-ci]
[![License][badge-license]][url-license]

<!-- Add a short demo screenshot/GIF here if applicable -->

## What This Is

Explain in 2–3 sentences what the project does, how it does it, and why that matters.
Skip the feature list here — the Features section below is for that.
Keep the "why" visible: what pain does this solve?

## Features

- **Feature name**: Brief description (one line per feature)
- Keep features scannable — avoid long paragraphs here
- Group by user goal, not by internal component

## Quick Start

```bash
# The minimum commands needed to get a running system
command-one
command-two
```

> Prerequisites: list what must be installed/configured before the quick start commands work.
> Example: Python 3.11+, Node.js 20+, ffmpeg, an API key, etc.

## Requirements

| Dependency | Version | Notes |
|---|---|---|
| python | ≥3.11 | Backend runtime |
| node | ≥20 | Frontend build |
| ffmpeg | latest | Media processing |
| ... | ... | ... |

## Installation

### Full setup

```bash
git clone <repo-url>
cd <repo-name>
# Backend
cd backend && uv sync --dev
# Frontend
cd ../web && npm install
```

### Configuration

```bash
# Describe where config lives and what the minimal config looks like
# Example:
cp config.example.json config.json
# Edit config.json and add your API key / credentials
```

## Usage

### Running locally

```bash
# Terminal 1 — backend
cd backend && uv run python -m locus.main

# Terminal 2 — frontend
cd web && npm run dev
```

Open [http://localhost:5173](http://localhost:5173) (or the port shown in your terminal).

### Production build

```bash
cd web && npm run build
# Backend serves the built SPA in production mode
```

## Architecture

Give a reader enough orientation to navigate the codebase:

```
top-level/
├── backend/          # What this does
│   └── src/locus/   # Key subdirectories and their purpose
├── web/             # What this does
│   └── src/        # Key subdirectories and their purpose
└── docs/           # What lives here
```

## Contributing

<!-- Adjust to match your project's contribution process -->

1. Fork and clone the repo
2. Create a branch: `git checkout -b feat/your-feature`
3. Run tests: `cd backend && uv run pytest` and/or `cd web && npm run test`
4. Lint: `uv run ruff check .` (backend) / `npm run lint` (web)
5. Commit using the commit convention: see [CLAUDE.md](CLAUDE.md) for the format
6. Open a PR

## License

[MIT](./LICENSE) — or whatever applies here.
