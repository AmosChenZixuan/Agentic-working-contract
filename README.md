# Agentic Working Contract (AWC)

Personal skills collection for AI coding agents. Built for **Claude Code**, also compatible with OpenCode, Codex, and others.

## Install

**All agents (recommended):**

```bash
npx skills add AmosChenZixuan/Agentic-working-contract -g -y
```

Bare skill names (`/agent-code`), works cross-platform, tracks updates via `npx skills update`.

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

1. **Confirm local skills are available.** Run `/agent-code --help` or equivalent.

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
| `/agent-scope` | Clarify ambiguous requirements into a concrete spec (file on disk) |
| `/agent-plan` | Break a spec into ordered, testable implementation slices (file on disk) |
| `/agent-code` | Execute a single slice, test-first, start to passing |
| `/grill-me` | Challenge your idea with design-decision questions |
| `/to-issues` | Convert specs or findings into agent-ready GitHub issues |
| `/clear-issues` | Autonomous issue lifecycle: investigate → implement → review → merge |
| `/c-review` | Lightweight PR review using `gh` CLI |
| `/c-simplify` | Review changed code for reuse, quality, efficiency; fix issues |
| `/coderabbit-review` | Run CodeRabbit AI review from the terminal |

---

## Two core workflows

The difference is **where intermediate results persist**:

| | Agentic Flow | Issue-Driven Flow |
|---|---|---|
| **Persistence** | Local files (specs, plans on disk) | Remote issue tracker (GitHub Issues) |
| **Traceability** | In repo — `git log` tells the story | In issues — comments, assignments, status transitions |
| **Best for** | Solo, fast iteration, local context | Team visibility, async handoff, audit trail |

### Agentic Flow

```
/agent-scope  →  /agent-plan  →  /agent-code
```

Spec and plan files saved in repo. Each phase gates the next. Use as the daily driver.

### Issue-Driven Flow

```
/grill-me  →  /to-issues  →  /clear-issues
```

Intermediate results live in GitHub Issues — visible, assignable, auditable. Use when collaborating or needing async handoff.

### Superpower Flow

```
superpowers/brainstorm  →  write a plan  →  execute the plan
```

Also file-based, but **heavier** — more constraints, higher token consumption. Use only when you genuinely don't know what to build yet. Requires `superpowers` installed.

### Quick decision

| Situation | Flow |
|-----------|------|
| Solo, fast iteration | Agentic Flow |
| Team, async handoff, audit trail | Issue-Driven Flow |
| Problem space is wide open, need creative exploration | Superpower Flow |
| Concrete bug fix or small feature | Skip to `/agent-code` |
| Just want code reviewed | `/c-review` or `/c-simplify` |
