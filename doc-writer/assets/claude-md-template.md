# {{ project-name }}

> Brief description: what the project is and what the primary user interface is.

## What This Project Is

**{{ project-name }}** does X. A [backend/framework] handles [concerns], and a [frontend] provides [UI surfaces].

## Commands

### Backend ({{ language }}, managed with {{ tool }})

```bash
cd backend
{{ install-command }}         # Install dependencies
{{ lint-command }}            # Lint
{{ test-command }}            # Run tests
{{ start-command }}          # Start server on :{{ port }}
```

### Frontend ({{ language }}/{{ framework }}, managed with {{ package-manager }})

```bash
cd web
npm install          # Install dependencies
npm run dev         # Start dev server on :5173
npm run build       # Production build
npm run test        # Run test suite
npm run lint        # Type-check / lint
```

## Architecture

```
Frontend (what, how served)
        ↕ REST/HTTP
Backend (language/framework)
    ├── Storage
    │   ├── SQLite — ...
    │   └── ... — ...
    └── ...
```

## Code Layout

**Backend** — `backend/src/{{ project }}/`

```
routers/     — one APIRouter per module; aggregated in main.py
models/      — request/response models
pipeline/    — {{ what each stage does }}
adapters/    — {{ platform-specific logic }}
db.py        — SQLite schema + query helpers
config.py    — settings persisted to ~/.{{ project }}/config.json
worker.py    — {{ job queue / dispatcher description }}
```

**Frontend** — `web/src/`

```
components/ui/  — primitive components
components/*/   — feature components (lists/, jobs/, layout/, settings/)
pages/          — route-level page components
lib/            — typed API client, types, utilities
app/            — router, language context
```

## Conventions

### Backend

- **Naming**: `snake_case` for files/functions/variables, `PascalCase` for classes and models
- **Models**: `{Operation}Request` / `{Operation}Response` suffix
- **Router pattern**: one `APIRouter` per file with explicit HTTP method decorators
- **DB**: `sqlite3` with `asynccontextmanager` wrapper; all queries use `?` placeholders
- **Imports**: stdlib → third-party → local
- **Errors**: `HTTPException(status_code=N, detail=...)` for HTTP errors; custom domain exceptions

### Frontend

- **Files**: `PascalCase` for components, `kebab-case` for utilities
- **Components**: props interface above component; `Record<Variant, string>` for variant maps
- **Handlers**: `handle{Action}`; event props use `on{Action}` prefix
- **State**: `useState` with `set` prefix; async operations use `let cancelled = false` guard in `useEffect`
- **Imports**: use `@/*` alias (`@/components/ui`, `@/lib/api`, `@/lib/types`)
- **Design tokens**: only tokens from `web/src/styles/app.css`

## Notes

- Technical specification, API contracts, and DB schema are in `docs/design-doc.md`.
- Active specs live in `docs/specs/`; plans in `docs/plans/`.
- Pre-commit hooks enforce linting and formatting. Run `pre-commit install` once after cloning.
- Config lives at `~/.{{ project }}/config.json`; runtime state at `~/.{{ project }}/`.
