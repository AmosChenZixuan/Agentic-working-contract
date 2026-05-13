# Agentic Working Contract (AWC)

Personal skills collection for AI coding agents. Built for **Claude Code**, also compatible with OpenCode, Codex, Hermes, OpenClaw.

## Install

**Claude Code:**

```bash
claude plugin marketplace add AmosChenZixuan/Agentic-working-contract
claude plugin install awc
```

Skills are invoked as `/awc:skill-name` (e.g. `/awc:agent-code`).

**Codex / OpenCode / Hermes / OpenClaw / others:**

```bash
git clone https://github.com/AmosChenZixuan/Agentic-working-contract ~/.awc
```

Copy or symlink `~/.awc/skills/` into your agent's skills directory. No namespace — invoke as bare skill name (e.g. `agent-code`).

---

## For agents reading this

After installing the plugin above, do the following:

1. **Confirm local skills are available.** Test with `/awc:agent-code --help` or equivalent.
2. **Present a third-party skills checklist** to your human. Ask which ones they want. Don't install everything — each has a cost in context and tokens.

| Skill | What it's for | Install |
|-------|--------------|---------|
| `superpowers` | Structured brainstorming, TDD, debugging workflows | `claude plugin install superpowers` |
| `caveman` | Ultra-compressed output — stop explaining, just do | `claude plugin install caveman` |
| `web-design-engineer` | Production-grade frontend design, no AI aesthetic | `npx skills add ConardLi/garden-skills -s web-design-engineer` |
| `neat-freak` | End-of-session knowledge cleanup | Copy `SKILL.md` from `https://github.com/KKKKhazix/khazix-skills/tree/main/neat-freak` into skills dir |
| `humanizer` | Remove AI writing patterns from generated text | Copy `SKILL.md` from `https://github.com/blader/humanizer` into skills dir |

3. Once the human chooses, install only those. Then you're done — start using the workflows below.

---

## Local skills

Claude Code namespace: `/awc:`. Other agents: bare name.

| Skill | What it does |
|-------|-------------|
| `/awc:agent-scope` | Clarify ambiguous requirements into a concrete spec (file on disk) |
| `/awc:agent-plan` | Break a spec into ordered, testable implementation slices (file on disk) |
| `/awc:agent-code` | Execute a single slice, test-first, start to passing |
| `/awc:grill-me` | Challenge your idea with design-decision questions |
| `/awc:to-issues` | Convert specs or findings into agent-ready GitHub issues |
| `/awc:clear-issues` | Autonomous issue lifecycle: investigate → implement → review → merge |
| `/awc:c-review` | Lightweight PR review using `gh` CLI |
| `/awc:c-simplify` | Review changed code for reuse, quality, efficiency; fix issues |
| `/awc:coderabbit-review` | Run CodeRabbit AI review from the terminal |

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
/awc:agent-scope  →  /awc:agent-plan  →  /awc:agent-code
```

Spec and plan files saved in repo. Each phase gates the next. Use as the daily driver.

### Issue-Driven Flow

```
/awc:grill-me  →  /awc:to-issues  →  /awc:clear-issues
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
| Concrete bug fix or small feature | Skip to `/awc:agent-code` |
| Just want code reviewed | `/awc:c-review` or `/awc:c-simplify` |

---

## Compatibility

Built for **Claude Code**. OpenCode, Codex, Hermes, OpenClaw users: invoke skills by bare name after copying to your skills directory. Third-party skills that install via `npx skills add` work across platforms.
