# Agent Skills

A collection of skills I use for Claude Code and Codex

## Skills

### Spec → Issues → Ship

Two workflows that cover the full lifecycle from idea to merged code.

**Single Session Development workflow** — takes a vague idea to shipped code in a single session:

```
agent-scope → agent-plan → agent-code
```

- **agent-scope** — Turns ambiguous requirements into a concrete spec with acceptance criteria. Writes to `docs/specs/`.
- **agent-plan** — Slices a spec into independently testable implementation units. Writes to `docs/plans/`.
- **agent-code** — Executes one slice: writes a failing test first, implements, verifies, and reports.

**Issue Driven workflow** — converts specs into a backlog and clears it autonomously:

```
agent-scope → to-issues → clear-issues
```

- **to-issues** — Decomposes a spec into an epic with dependency-ordered sub-issues, or creates standalone issues from conversation findings. Each issue is structured so agents can pick it up without reading the full spec.
- **clear-issues** — Orchestrates the full issue lifecycle: triage (epics, layers, dependencies) → grill → dispatch to worktree → implement (TDD + simplify) → review → CI → merge → close. Respects layer ordering and auto-unblocks the next layer after each merge.

`to-issues` produces the backlog, `clear-issues` consumes it. Issues created by `to-issues` carry structural metadata (`Parent:`, `Layer:`, `Depends on:`) that `clear-issues` uses during triage to determine execution order.

Start with `/agent-scope` when the requirement is unclear, then `/to-issues` to create the backlog, then `/clear-issues` to execute it.

### Supporting Skills

- **grill-me** — Rigorous one-at-a-time questioning to stress-test assumptions until a shared understanding is reached.
- **[frontend-ui-ux](https://github.com/code-yeongyu/oh-my-openagent/blob/dev/src/features/builtin-skills/frontend-ui-ux/SKILL.md)** — Builds production-grade UI with a strong aesthetic point of view.
- **[ui-ux-pro-max](https://github.com/nextlevelbuilder/ui-ux-pro-max-skill)** — UI/UX design intelligence: 67 styles, 96 palettes, 57 font pairings, 13 tech stacks. Use for landing pages, dashboards, mobile apps, and component-level design decisions.

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
