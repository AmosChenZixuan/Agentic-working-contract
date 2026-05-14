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

2. **Check what's already installed.** Run `npx skills ls -g` and scan `~/.claude/skills/`. Remove any already-installed skills from the checklist below — don't ask the human about things they already have.

3. **Present an interactive checklist** (max 4 options per question — split into 2 questions if needed).
   Use your platform's Q&A tool: Claude Code → `AskUserQuestion` (multiSelect: true), OpenCode → `question`, Hermes → `clarify`.
   If no Q&A tool, post as markdown and ask the human to reply.

   **Question 1 — Workflow & output:**
   - superpowers — brainstorming, TDD, debugging
     install: `npx skills add obra/superpowers -g -y`
   - caveman — ultra-compressed output
     install: `npx skills add JuliusBrussee/caveman -g -y`
   - web-design-engineer — production frontend design
     install: `npx skills add ConardLi/garden-skills -s web-design-engineer -g -y`

   **Question 2 — Cleanup & style:**
   - neat-freak — end-of-session knowledge cleanup
     install: `npx skills add KKKKhazix/khazix-skills -s neat-freak -g -y`
   - humanizer — remove AI writing patterns
     install: `npx skills add blader/humanizer -s humanizer -g -y`

4. Install only the skills the human picked.

5. **Tell the human how to update:**
   - `npx skills ls -g` — check what's installed and if updates are available
   - `npx skills update` — update all skills
   - `npx skills update <name>` — update a single skill

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
