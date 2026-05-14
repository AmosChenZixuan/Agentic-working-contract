# Skill Fallback Matrix

shipit probes available skills and MCP servers at session start. For each phase, use the highest available tier.

## Per-phase matrix

| Phase | Preferred | Local fallback | Degrade (inline) |
|---|---|---|---|
| Branch / worktree setup | `superpowers:using-git-worktrees` | `git worktree add` inline | `git checkout -b <feature-slug>` (feature branch fallback) |
| Clarify intent | `superpowers:brainstorming` | `grill-me` | One question per turn until problem / success / out-of-scope / constraints all answered |
| Plan writing | `superpowers:writing-plans` | — | Slice list with closing boundary test per slice |
| Smith's TDD | `superpowers:test-driven-development` | — | Write failing test → confirm it fails for the right reason → impl → run → verify output |
| Smith's verify | `superpowers:verification-before-completion` | — | Run full suite; read output; claim success only after evidence |
| Hawk's review | platform-native `/review` (Claude Code) or equivalent (Codex) | `c-simplify` | Inline heuristics — correctness, safety, regression, perf, style |
| Scout's AC verify (web-ui) | playwright MCP | — | Manual walkthrough requested to user |
| Scout's AC verify (http-api) | Bash + curl/fetch (always available) | — | — |
| Scout's AC verify (cli) | Bash + shell exec (always available) | — | — |
| Scout's AC verify (library) | Test runner with import + call (always available) | — | — |
| Post-ship doc cleanup | `neat-freak` | — | Inline doc + memory update |
| Commit message | `caveman:caveman-commit` | — | Inline Conventional Commits |

## Bootstrap probe (Phase 0)

At session start, run **both** sub-probes:

### 0a. Branch / worktree probe

```bash
git rev-parse --abbrev-ref HEAD
git worktree list
```

- On `main` or `master` → block. Prompt user:
  > "shipit refuses to work on `<branch>`. Create a worktree (preferred) or feature branch? [worktree / branch / abort]"
- On long-lived shared branch (`develop`, `staging`, `release/*`) → warn and confirm.
- On a feature branch or in a worktree → proceed.

If `superpowers:using-git-worktrees` is available, defer the worktree creation to it. Otherwise inline:

```bash
git worktree add ../<repo-name>-<feature-slug> -b <feature-slug>
```

Then `cd` into the worktree and continue.

### 0b. Skill / MCP probe

Inspect the available skill list and configured MCP servers. Output the resolved tier per phase to the user:

```
shipit bootstrap probe:
  Branch / worktree: <tier> — <current branch or worktree path>
  Clarify intent:    <tier> — <skill name or "inline">
  Plan writing:      <tier> — <skill name or "inline">
  Smith's TDD:       <tier> — <skill name or "inline">
  Smith's verify:    <tier> — <skill name or "inline">
  Hawk's review:     <tier> — <skill name or "inline">
  Scout's AC verify: deferred until feature type is known (web-ui requires playwright MCP)
  Post-ship doc:     <tier> — <skill name or "inline">
  Commit message:    <tier> — <skill name or "inline">
```

For each `degraded` phase, prompt the user **once per session**:

> "Install `<skill-name>` for better `<phase>`? Currently degraded to inline. [y / n / skip-all]"

- `y` → main agent runs the install command (see below)
- `n` → proceed with degraded tier for this session
- `skip-all` → don't ask again this session, proceed with whatever is available

shipit **never** auto-installs.

## Install commands

| Skill / tool | Install path |
|---|---|
| superpowers (using-git-worktrees, brainstorming, writing-plans, test-driven-development, verification-before-completion) | `npx skills add obra/superpowers` — refer to upstream README for per-agent native install path |
| neat-freak | `npx skills add KKKKhazix/khazix-skills` |
| caveman (caveman-commit) | `npx skills add JuliusBrussee/caveman` |
| coderabbit-review | Already local in this repo; no install needed |
| c-simplify, c-review, grill-me | Already local in this repo; no install needed |
| playwright MCP | Not a skill. Add via the agent's MCP configuration. Example for Claude Code: `claude mcp add playwright npx -- @playwright/mcp@latest`. Defer to user; shipit does not configure MCP. |

When in doubt about the install URL or path, point the user at the upstream guide referenced in this repo's README rather than guessing.

## Feature-type classification for Scout

When dispatching Scout, classify the feature so Scout picks the right verification tool:

- **web-ui** — React, Vue, frontend, browser-rendered. Requires playwright MCP for preferred tier.
- **http-api** — REST, GraphQL, RPC. Bash + curl/fetch always works.
- **cli** — command-line tool, script. Bash + shell exec always works.
- **library** — importable module, SDK. Test runner with import + call always works.

Mixed features (e.g., API with UI front): main agent passes `feature_type: [web-ui, http-api]` to Scout, and Scout verifies each AC against the appropriate surface.
