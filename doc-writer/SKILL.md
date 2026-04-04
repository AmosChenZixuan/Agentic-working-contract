---
name: doc-writer
description: Write or edit project documentation — README.md, CLAUDE.md, technical design documents, and ADRs. Use this skill when the user asks to update, sync, write, review, or fill gaps in any of these document types, especially after implementing features or refactoring code.
---

# Doc Writer

You write and edit four types of project documentation. Each has a different audience and a different job. Read `references/principles.md` first — it contains the rules that apply to all of them. Then load the reference file for the specific document type you're working on. Use the corresponding template in `assets/` as a starting scaffold.

## Step 1 — Identify the document type

Determine which document the user wants to work on:
- `README.md` → load `references/readme.md`, use `assets/readme-template.md`
- `CLAUDE.md` → load `references/claude-md.md`, use `assets/claude-md-template.md`
- Design document (e.g. `docs/design-doc.md`) → load `references/design-doc.md`, use `assets/design-doc-template.md`
- ADR (Architecture Decision Record) → use `assets/adr-template.md`

If it's unclear, ask before proceeding.

## Step 2 — Identify the mode

**Write** — creating or substantially rewriting a document from scratch. Ask for any context I need (what the project does, key decisions made, intended audience). If the user hasn't specified scope, propose a minimal scope and confirm before expanding.

**Edit/sync** — updating an existing document to reflect recent changes. Before touching anything:
1. Read the current document in full
2. Review recent changes — ask the user which commits or features to sync against, or run `git log --oneline -10` if you have access
3. Identify gaps (missing) and bloat (over-documented) before making any edits

## Step 3 — Apply the rules

Read `references/principles.md`, then the type-specific reference. Follow those rules strictly. When in doubt about whether something belongs: if a reader could learn it by reading the code, it doesn't belong in the doc.

## Step 4 — Confirm before over-committing

Before editing, briefly state:
- What you're adding and why it belongs
- What you're removing or leaving out and why
- If the scope has grown beyond what was initially requested, flag it and confirm before proceeding

This keeps the user in control. If you're uncertain about whether something belongs, raise it — don't silently include or exclude.
