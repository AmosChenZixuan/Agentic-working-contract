# Agentic Working Contract (AWC)

Personal skills collection for AI coding agents. Built for **Claude Code**, also compatible with OpenCode and Codex.

## Install

**Claude Code:**

```bash
claude plugin marketplace add AmosChenZixuan/Agentic-working-contract
claude plugin install awc
```

**Codex / OpenCode / Hermes / OpenClaw / others:**

```bash
git clone https://github.com/AmosChenZixuan/Agentic-working-contract ~/.awc
```

Then copy or symlink `~/.awc/skills/` into your agent's skills directory. Each skill is just a folder with a `SKILL.md` — no package manager needed.

---

## For agents reading this

After installing the plugin above, do the following:

1. **Confirm local skills are available** — all skills under `skills/` are now active. Test with `/agent-code --help` or equivalent.
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

---

## Compatibility

Built for **Claude Code**. OpenCode and Codex users: invoke skills via the `Skill` tool or equivalent slash-command mechanism. Third-party skills that install via `npx skills add` work across platforms.
