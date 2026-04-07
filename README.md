# Agent Skills

A collection of skills I use for Claude Code and Codex

## Skills

### Agent Development Workflow

Three skills that work together to take a vague idea to shipped code:

```
agent-scope → agent-plan → agent-code
```

- **agent-scope** — Turns ambiguous requirements into a concrete spec with acceptance criteria. Writes to `docs/specs/`.
- **agent-plan** — Slices a spec into independently testable implementation units. Writes to `docs/plans/`.
- **agent-code** — Executes one slice: writes a failing test first, implements, verifies, and reports.

Start with `/agent-scope` when the requirement is unclear or the approach is undecided.

### Other Skills

- **grill-me** — Rigorous one-at-a-time questioning to decompose an idea and stress-test assumptions until a shared, well-defined understanding is reached.
- **to-issue** — Converts conversations into GitHub issues. Detects issue type from context and fills the appropriate template.
- **doc-writer** — Writes and edits `README.md`, `CLAUDE.md`, and design documents.
- **[frontend-ui-ux](https://github.com/code-yeongyu/oh-my-openagent/blob/dev/src/features/builtin-skills/frontend-ui-ux/SKILL.md)** — Builds production-grade UI with a strong aesthetic point of view.
- **[ui-ux-pro-max](https://github.com/nextlevelbuilder/ui-ux-pro-max-skill)** — UI/UX design intelligence: 67 styles, 96 palettes, 57 font pairings, 13 tech stacks. Use for landing pages, dashboards, mobile apps, and component-level design decisions.

- [**Superpowers Subset**](https://github.com/obra/superpowers)
    - **systematic-debugging** — Use when encountering any bug, test failure, or unexpected behavior. Structured root-cause tracing and defense-in-depth.
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
