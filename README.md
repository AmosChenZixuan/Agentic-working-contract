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

- **doc-writer** — Writes and edits `README.md`, `CLAUDE.md`, and design documents.
- **frontend-ui-ux** — Builds production-grade UI with a strong aesthetic point of view.

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
