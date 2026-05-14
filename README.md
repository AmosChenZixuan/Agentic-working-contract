# Agentic Working Contract (AWC)

Personal skills collection for AI coding agents. Built for **Claude Code**, also compatible with OpenCode, Codex, and others.

## Install

**All agents (recommended):**

```bash
npx skills add AmosChenZixuan/Agentic-working-contract -g -y
```

Bare skill names (`/shipit`), works cross-platform, tracks updates via `npx skills update`.

**Claude Code only (marketplace):**

```bash
claude plugin marketplace add AmosChenZixuan/Agentic-working-contract
claude plugin install awc
```

Note: marketplace adds a `/awc:` prefix to skill names.

**Manual (any agent):**

```bash
git clone https://github.com/AmosChenZixuan/Agentic-working-contract /tmp/awc
cp -r /tmp/awc/skills/* ~/.agent/skills/
```

### Install third-party skills
After install, tell your agent:
```
read https://github.com/AmosChenZixuan/Agentic-working-contract/blob/main/README.md and install the third-party skills
```

---

## For agents reading this

You just installed a set of slash-command skills. Now do the following:

1. **Confirm local skills are available.** Run `/shipit --help` or equivalent.

2. **Check what's already installed.** Run `npx skills ls` and `npx skills ls -g`, scan both `.agent/skills/` and `~/.agent/skills/`. Remove any already-installed skills from the checklist below — don't ask the human about things they already have.

3. **Present an interactive checklist** (max 4 options per question — split into more questions if needed).
   Use your platform's Q&A tool: Claude Code → `AskUserQuestion` (multiSelect: true), OpenCode → `question`, Hermes → `clarify`.
   If no Q&A tool, post as markdown and ask the human to reply.

   For each skill the human picks, determine install scope:
   - **Has per-agent native install** — check the upstream install guide, then
     detect the current agent and use its native path
     (e.g. `claude plugin install` for Claude Code, `npx skills add -a cursor` for Cursor).
     If native path unavailable, ask: "install globally (all projects) or per-project (this directory only)?"
   - **npx-only** — no marketplace, only `npx skills add` or file copy.
     Ask the human: "install globally (-g, all projects) or per-project (current directory only)?"

   **Question 1 — Workflow & output:**
   - superpowers — brainstorming, TDD, debugging
     guide: https://github.com/obra/superpowers/blob/main/README.md
   - caveman — ultra-compressed output
     guide: https://github.com/JuliusBrussee/caveman/blob/main/INSTALL.md
   - web-design-engineer — production frontend design
     guide: https://github.com/ConardLi/garden-skills/blob/main/README.md

   **Question 2 — Cleanup & style:**
   - neat-freak — end-of-session knowledge cleanup
     guide: https://github.com/KKKKhazix/khazix-skills/blob/main/README.md
   - humanizer — remove AI writing patterns
     guide: https://github.com/blader/humanizer/blob/main/README.md

4. Install only the skills the human picked, using the scope they chose.

5. **Tell the human how to update:**
   - **npx-installed skills:** `npx skills ls` (per-project) / `npx skills ls -g` (global) to check; `npx skills update` to update all
   - **Per-agent native install** (e.g. `claude plugin install`): use that agent's native update mechanism — run `claude plugin update`, check the upstream guide, or re-run the install command
   - When in doubt, re-run the original install command for that skill

---

## Local skills

All invoked as slash commands. Skills are auto-discovered from the `skills/` directory.

| Skill | What it does |
|-------|-------------|
| `/shipit` | Full-cycle PR delivery — main agent owns design + AC + planning + arbitration; spawns **Smith** (white-box implementer), **Scout** (black-box AC verifier), and **Hawk** (white-box reviewer) in a bounded critique loop on a worktree or feature branch; output is a ship-ready PR (never auto-merged) |
| `/grill-me` | Challenge your idea with design-decision questions |
| `/to-issues` | Convert specs or findings into agent-ready GitHub issues |
| `/clear-issues` | Autonomous issue lifecycle: investigate → implement → review → merge |
| `/c-review` | Lightweight PR review using `gh` CLI |
| `/c-simplify` | Review changed code for reuse, quality, efficiency; fix issues |
| `/coderabbit-review` | Run CodeRabbit AI review from the terminal |

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
| Just want code reviewed | `/c-review` or `/c-simplify` |
