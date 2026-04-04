# CLAUDE.md Rules

**Template:** `assets/claude-md-template.md` — start here for new CLAUDE.md files. Fill in the `{{ placeholders }}` and delete sections that don't apply.

**Audience:** AI agents working in this repository. They have no memory of prior sessions and need to know how to operate, not what exists.

## What belongs

- Commands to run the project (dev, test, build)
- Conventions an agent must follow that aren't obvious from the code (naming, patterns, constraints)
- Where to look for things — key *folders* and what they contain
- Rules that apply to *all* agents working here, regardless of task

## What doesn't belong

- Enumerations of components, routes, columns, or features — agents read the code
- Anything that changes with every feature addition (a list of UI components, API routes, DB columns)
- Rationale for decisions — that's the design doc
- Project status or recent changes — agents run git log

## The stability test

If a line in CLAUDE.md would need to be updated after most feature PRs, it doesn't belong there. CLAUDE.md should describe how the repo works, not what it currently contains.

## Format

Keep it scannable. Use tables for lookup-style information (e.g. component variants). Use code blocks for commands. Short prose for conventions. Agents skim — they don't read.
