---
name: doc-writer
description: Write or edit project documentation — README.md, CLAUDE.md, and technical design documents. Use this skill when the user asks to update, sync, write, review, or fill gaps in any of these three document types, especially after implementing features or refactoring code.
---

# Doc Writer

You write and edit three types of project documentation. Each has a different audience and a different job. Read `references/principles.md` first — it contains the rules that apply to all three. Then load the reference file for the specific document type you're working on.

## Step 1 — Identify the document type

Determine which document the user wants to work on:
- `README.md` → load `references/readme.md`
- `CLAUDE.md` → load `references/claude-md.md`
- Design document (e.g. `docs/design-doc.md`) → load `references/design-doc.md`

If it's unclear, ask before proceeding.

## Step 2 — Identify the mode

**Write** — creating or substantially rewriting a document from scratch. Ask for any context you need (what the project does, key decisions made, intended audience).

**Edit/sync** — updating an existing document to reflect recent changes. Before touching anything:
1. Read the current document in full
2. Review recent changes — ask the user which commits or features to sync against, or run `git log --oneline -10` if you have access
3. Identify gaps (missing) and bloat (over-documented) before making any edits

## Step 3 — Apply the rules

Read `references/principles.md`, then the type-specific reference. Follow those rules strictly. When in doubt about whether something belongs: if a reader could learn it by reading the code, it doesn't belong in the doc.

## Step 4 — Show your reasoning

Before editing, briefly state:
- What you're adding and why it belongs
- What you're removing or leaving out and why

This keeps the user in control of what gets documented.
