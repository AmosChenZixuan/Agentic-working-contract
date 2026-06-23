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
| `/shipit` | Take one agent-ready unit → a review-ready PR. Main agent plans, codes, and commits on a worktree or feature branch; reviews each revision through a sequential whitebox (`razor-code`) then blackbox (live smoke / visual) loop until clean; opens a non-draft PR with a reflection comment. One unit per run; never merges — the human reviews and merges |
| `/grill-me` | Challenge your idea with design-decision questions |
| `/to-issues` | Convert specs or findings into agent-ready GitHub issues |
| `/razor` | Guard a design before building — reconstructs the user's true need (≠ what they asked for), derives the smallest design that meets it, and names the rest as over-design. Pre-implementation only |
| `/razor-code` | Cut cruft from code already written — flags over-engineering, needless complexity & duplication, dead-weight comments, and low-value tests, while keeping what protects correctness & safety. Post-implementation counterpart to `/razor` |

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

### The AWC chain

```
/grill-me  →  /to-issues  →  /shipit
```

The backbone of the contract. Sharpen the idea into design decisions, file it as agent-ready issues, then ship one issue at a time into a review-ready PR.

- **`/grill-me`** — pressure-test the idea one decision at a time until the goal is fully resolved.
- **`/to-issues`** — turn the resolved spec (or a conversation) into agent-ready GitHub issues: spikes, epics, and 1:1-with-a-PR issues.
- **`/shipit`** — take **one** agent-ready unit and deliver **one** review-ready PR. The main agent plans, codes, and commits (incremental, never amended) on a worktree or feature branch — never on `main`. Each revision runs a sequential review loop: whitebox (`razor-code`) clears, then blackbox (live smoke / visual, only when an AC needs the feature running). The loop repeats until both reviewers return zero blockers, then shipit opens a non-draft PR and posts a session reflection as a comment. It never merges and never closes the issue — the human reviews, merges, and closes.

Each link stands alone: `/shipit` takes any agent-ready spec or clear request, building one inline (`grill-me` → spec → `razor`) if it isn't ready yet.

### Superpower Flow (alternative)

```
superpowers/brainstorm  →  write a plan  →  execute the plan
```

File-based, **heavier** — more constraints, higher token consumption. Use only when the problem space is wide open. Requires `superpowers` installed; shipit invokes parts of superpowers internally when available.

### Quick decision

| Situation | Use |
|-----------|-----|
| Idea needs sharpening before you build | `/grill-me` |
| Resolved work to capture as trackable issues | `/to-issues` |
| One agent-ready unit → a review-ready PR | `/shipit` |
| Problem space is wide open, need creative exploration | Superpower Flow |
| Worried a design is overkill before building | `/razor` |
| Code is written and you want it leaner | `/razor-code` |