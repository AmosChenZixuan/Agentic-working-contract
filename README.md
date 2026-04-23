# Agent Skills

A collection of skills I use for Claude Code and Codex.

## Workflow Comparison

Three ways to go from idea to shipped code. Pick the one that fits the job.

### At a Glance

| | feature-dev | agentic chain | Issue-driven |
|---|---|---|---|
| **Invoke** | `/feature-dev` | `/agent-scope` → `/agent-plan` → `/agent-code` | `/grill-me` → `/to-issues` → `/clear-issues` |
| **Best for** | Quick features in a familiar codebase | Cross-session work that needs rigor | Large features, multi-day work, team backlogs |
| **Artifacts** | None (in-context only) | Spec file + plan file | Spec file + GitHub issues (epic + subs) |
| **Survives context loss** | No | Yes (files on disk) | Yes (files + GitHub) |
| **Scoping** | Clarifying questions after parallel exploration | Adversarial questioning via grill-me | Adversarial questioning via grill-me |
| **Architecture** | 3 competing designs (minimal / clean / pragmatic) | Single plan sliced into testable units | Same plan, decomposed into issues |
| **Implementation** | Single pass in main context | TDD per slice, explicit scope check | TDD in isolated git worktrees |
| **Review** | 3 parallel reviewers at the end (confidence ≥80) | Two-stage per slice (spec compliance → code quality) | Two-stage per issue + CI gate before merge |
| **Parallelism** | Parallel exploration + architecture + review | Sequential slices (or parallel via subagent-driven-dev) | Parallel worktrees across independent issues |
| **Isolation** | None (works in current branch) | Optional (git worktrees) | Required (one worktree per issue) |

### Cross-Session Development Workflow
```
agent-scope → agent-plan → agent-code
```

- **agent-scope** — Adversarial scoping. Writes a concrete spec with acceptance criteria to `docs/specs/`.
- **agent-plan** — Slices the spec into independently testable units with dependency ordering. Writes to `docs/plans/`.
- **agent-code** — Executes one slice: failing test first, implement, verify, scope-check the diff, report.

### Issue-Driven Development Workflow
```
grill-me → to-issues → clear-issues
```

- **to-issues** — Decomposes a spec into an epic with dependency-ordered sub-issues. Each issue carries structural metadata (`Parent:`, `Layer:`, `Depends on:`) so agents can pick it up cold.
- **clear-issues** — Orchestrates: triage → grill → dispatch to worktree → implement (TDD) → review → CI verification → squash merge → close. Respects layer ordering and auto-unblocks the next layer after each merge.

### When to Use What

| Scenario | Use |
|---|---|
| Small feature, familiar code, single sitting | `feature-dev` |
| Unclear requirements that need stress-testing | `agent-scope` → decide next step |
| Medium feature, want TDD and scope discipline | `agent-*` chain |
| Large feature spanning days or multiple agents | Issue-driven |
| Clearing a backlog of known work | `clear-issues` |
| Need competing architecture options | `feature-dev` (3 architect agents) |
| Need to explore an unfamiliar codebase | `feature-dev` Phase 2 (parallel explorers) |

### Supporting Skills

- **grill-me** — Rigorous one-at-a-time questioning to stress-test assumptions until a shared understanding is reached.
- **[ui-ux-pro-max](https://github.com/nextlevelbuilder/ui-ux-pro-max-skill)** — UI/UX design intelligence: 67 styles, 96 palettes, 57 font pairings, 13 tech stacks. Use for landing pages, dashboards, mobile apps, and component-level design decisions.
- **[web-design-engineer](https://github.com/ConardLi/web-design-skill)** — High-quality visual web artifacts: landing pages, dashboards, interactive prototypes, HTML slide decks, data visualizations.
- **c-review** — Inspired by Claude Code's review command. Concise, actionable code review comments.
- **c-simplify** — Inspired by Claude Code's simplify command. Reviews changed code for reuse, quality, and efficiency, then fixes issues found.
- **coderabbit-review** — Inspired by CodeX review. Autonomous PR feedback with code quality checks and fix-review cycles.

- [**Superpowers Subset**](https://github.com/obra/superpowers)
    - **systematic-debugging** — Structured root-cause tracing and defense-in-depth for bugs and test failures.
    - **subagent-driven-development** — Coordinates multiple agents working on independent tasks in a single session.
    - **dispatching-parallel-agents** — Launches 2+ agents simultaneously for tasks with no shared state or sequential dependencies.
    - **using-git-worktrees** — Creates isolated git worktrees for feature work that needs to stay clean of the main workspace.

## Install

Clone once, then link into Codex:

```bash
git clone https://github.com/AmosChenZixuan/Agentic-working-contract.git ~/.claude/skills
ln -s ~/.claude/skills ~/.codex/skills/claude-user-skills
```

## Usage

| Tool | Invoke |
|------|--------|
| Claude Code | `/skill-name` |
| Codex | `$skill-name` |

Both load the skill's `SKILL.md` and follow its instructions for that session.

## Adding a Skill

Create a directory under `skills/` with a `SKILL.md` file. The frontmatter `name` and `description` fields control how Claude Code surfaces it.
