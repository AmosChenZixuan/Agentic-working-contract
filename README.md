# Agentic Working Contract (AWC)

Personal skills collection for AI coding agents. Built for **Claude Code**, also compatible with OpenCode, Codex, and others.

## Install

**AWC's bundled skills (recommended):**

```bash
npx skills add AmosChenZixuan/Agentic-working-contract -g -y
```

Bare skill names (`/shipit`), works cross-platform, tracks updates via `npx skills update`.

**Third-party skills or plugins:**

Tell your agent:

```
read https://github.com/AmosChenZixuan/Agentic-working-contract/blob/main/INSTALL.md and follow it
```

The harness walks you through picking skills (superpowers, caveman, etc.) and Claude plugins (feature-dev, claude-hud, etc.) and installs only what you choose.

---

## Bundled skills


| Skill | What it does |
|-------|-------------|
| `/shipit` | Full-cycle PR delivery — main agent owns design + AC + planning + arbitration; spawns **Smith** (white-box implementer), **Scout** (black-box AC verifier), and **Hawk** (white-box reviewer) in a bounded critique loop on a worktree or feature branch; output is a ship-ready PR (never auto-merged) |
| `/grill-me` | Challenge your idea with design-decision questions |
| `/to-issues` | Convert specs or findings into agent-ready GitHub issues |
| `/clear-issues` | Autonomous issue lifecycle: investigate → implement → review → merge |
| `/c-simplify` | Review changed code for reuse, quality, efficiency; fix issues |
| `/coderabbit-review` | Run CodeRabbit AI review from the terminal |

---

## Third-party skills

Installable via the harness (`INSTALL.md`). Cross-agent.

| Skill | What it does |
|-------|-------------|
| `superpowers` | brainstorming, TDD, debugging |
| `caveman` | ultra-compressed output |
| `web-design-engineer` | production frontend design |
| `neat-freak` | end-of-session knowledge cleanup |
| `humanizer` | remove AI writing patterns |
| `handoff` | compact the conversation into a handoff doc for another agent |
| `teach` | teach a skill or concept across sessions, using the current dir as state |
| `ponytail` | flag & reduce over-engineering in diffs and repos |

---

## Plugins

Installable via the harness (`INSTALL.md`)

### Claude Code

| Plugin | What it does |
|--------|-------------|
| `feature-dev` | guided feature development with codebase understanding |
| `pr-review-toolkit` | comprehensive PR review using specialized agents |
| `chrome-devtools-mcp` | Chrome DevTools via MCP for debugging and browser automation |
| `claude-hud` | statusline for Claude Code |
| `claude-md-management` | audit & maintain CLAUDE.md files |
| `commit-commands` | git commit / push / PR commands |
| `context7` | up-to-date library docs via MCP |
| `skill-creator` | create, edit, and eval skills |

---

## Core workflows

Two orthogonal axes: **how the work is partitioned** (context-boundary vs task-boundary) and **where intermediate state lives** (local files vs issue tracker).

| | Shipit Flow | Issue-Driven Flow | Superpower Flow |
|---|---|---|---|
| **Partition** | Context-boundary — Smith keeps full feature context end-to-end; parallel critics Scout + Hawk post-coding | Task-boundary — each subagent gets a slice via the issue tracker | Task-boundary — phases handed off via spec / plan files |
| **Persistence** | Spec + AC + findings on disk; PR in git | GitHub Issues + comments | Spec + plan files in repo |
| **Critique** | Parallel Scout (black-box AC) + Hawk (white-box review) with bounded pushback per finding | Reviews via PR comments / linked issues | Optional review skills, sequential |
| **Best for** | Single PR-sized feature; minimal handoff loss; structured critique | Team visibility, async handoff, audit trail | Open-ended exploration where the problem space isn't yet defined |

### Shipit Flow

```
/shipit  →  worktree/branch  →  design + AC  →  Smith (impl+test+refactor)  →  Scout ∥ Hawk critique  →  ship-ready PR
```

Main agent writes the spec + AC on a fresh worktree or feature branch (never on `main`), then dispatches Smith end-to-end. Scout and Hawk run in parallel after Smith submits; findings use a structured schema with a per-finding 3-pushback budget. Never auto-merges. Use as the daily driver for PR-sized features.

### Issue-Driven Flow

```
/grill-me  →  /to-issues  →  /clear-issues
```

Intermediate results live in GitHub Issues — visible, assignable, auditable. Use when collaborating or needing async handoff.

### Superpower Flow

```
superpowers/brainstorm  →  write a plan  →  execute the plan
```

File-based, **heavier** — more constraints, higher token consumption. Use only when the problem space is wide open. Requires `superpowers` installed. shipit invokes parts of superpowers internally when available.

### Quick decision

| Situation | Flow |
|-----------|------|
| Single PR-sized feature, fast iteration | Shipit Flow |
| Team, async handoff, audit trail | Issue-Driven Flow |
| Problem space is wide open, need creative exploration | Superpower Flow |
| Concrete bug fix or small feature | `/shipit` (still — it handles trivial features too, just lighter critique) |
| Just want code reviewed | `/c-simplify` |