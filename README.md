# Agentic Working Contract (AWC)

Personal skills collection for AI coding agents. Built for **Claude Code**, also compatible with OpenCode, Codex, and others.

## Install

**Claude Code:**

```bash
claude plugin marketplace add AmosChenZixuan/Agentic-working-contract
claude plugin install awc
```

**Codex / OpenCode / Hermes / OpenClaw / Others:**

```bash
git clone https://github.com/AmosChenZixuan/Agentic-working-contract ~/.awc
```

Then copy `~/.awc/skills/` into your agent's skills directory.


### Install Third-party Skills
After install, tell your agent: 
```
install the third-party skills recommended by AWC
```
It will read this README and walk through the rest.

---

## For agents reading this

You just installed a set of slash-command skills. Now do the following:

1. **Confirm skills are available.** Run `/agent-code --help` or equivalent.

2. **Present an interactive checklist** (max 4 options per question — split into 2 questions if needed).
   Use your platform's Q&A tool: Claude Code → `AskUserQuestion` (multiSelect: true), OpenCode → `question`, Hermes → `clarify`.
   If your platform has no Q&A tool (Codex, OpenClaw), post as markdown and ask the human to reply.

   **Question 1 — Workflow & output:**
   - superpowers — brainstorming, TDD, debugging workflows
     install: `claude plugin install superpowers`
   - caveman — ultra-compressed output, stop explaining just do
     install: `claude plugin marketplace add JuliusBrussee/caveman && claude plugin install caveman`
   - web-design-engineer — production frontend design, no AI aesthetic
     install: `npx skills add ConardLi/garden-skills -s web-design-engineer -g -y`

   **Question 2 — Cleanup & style:**
   - neat-freak — end-of-session knowledge cleanup
     install: clone `KKKKhazix/khazix-skills`, copy the `neat-freak/` directory into `~/.claude/skills/`
   - humanizer — remove AI writing patterns from generated text
     install: clone `blader/humanizer`, copy the skill directory into `~/.claude/skills/`

3. Install only the skills the human picked. Then done — start using the workflows below.

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
