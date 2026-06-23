---
name: to-issues
description: Convert a conversation or spec into agent-ready GitHub issues. Use when the user invokes /to-issues or /to-issue, says "file this", "break this into issues", "create issues from this", "capture this as backlog", or when conversation surfaces a bug, design gap, deferred work, or unverified idea worth tracking. Outputs spikes (time-boxed investigations), epics (sub-issue roadmaps), and issues (1:1 with a PR). Each filed issue is consumed by /clear-issues for grill → dispatch → implement. This is the counterpart to /clear-issues — it produces the backlog that /clear-issues consumes.
---

# to-issues

Convert a conversation into one or more **agent-ready** issues. Optionally wrap them in an **epic** (parent tracker). Optionally precede them with a **spike** (time-boxed investigation).

## Outcome types

| Type | What it is | Closes when |
|------|-----------|-------------|
| **issue** | Feature / bug / refactor, agent-ready spec | The PR is merged. **1 issue = 1 PR, always.** |
| **epic** | Tracker for N sub-issues with the dependency graph | All sub-issues are closed. Specs live in the sub-issues, not the epic. |
| **spike** | Time-boxed investigation | A closing comment links the deliverable: a handoff doc, a POC branch, or a new set of filed issues. |

**Never bundle fix + cleanup, or feature + refactor, into one issue.** Different risk profiles, different PRs, different reviews.

**Use a spike when you don't know yet.** File a spike instead of guessing the spec.

## Decompose (3 passes)

For a single finding the passes collapse to one issue — decomposition only earns its keep once there's more than one independently useful unit.

1. **Seams** — find independent boundaries (backend/frontend, schema/logic, mechanical/behavioral, diagnosis/fix). For bugs, each independent root cause is its own seam.
2. **Smallest useful unit** — merge things that aren't testable alone; split things that are independently useful. Never split by file.
3. **Dependency graph** — layer from leaves up. Same layer + no shared dependency = parallelizable. Each issue mergeable to master on its own.

## Lock spec and dependencies before filing

Before drafting Scope, scan the real files the work touches so issues name actual modules and types, not "follow existing patterns."

Freeze before any `gh issue create`:

- **Spec** — Goal, Scope (real files / types / APIs), Out of scope, Acceptance (objectively verifiable, never "works correctly").
- **Dependencies** — every `Depends on: #NN` edge resolves to a real issue number, not a placeholder.

If the spec is still moving, file a spike — not a moving issue. Changes after filing are a new issue or an explicit amend, not a silent edit.

## Propose → confirm → file

Show a numbered list (title + 1-line summary + labels) and an ASCII dependency diagram. Note what is parallelizable. **Wait for confirmation** — the user may merge, split, reorder, or drop.

Then `gh issue create` per issue. Match the repo's label set and title prefix convention (`gh label list`, `gh issue list --limit 10`). After all sub-issues are filed, update the epic body to replace placeholder `#NN` with real numbers.

## Templates

| Type | Template |
|------|----------|
| Epic | `references/epic.md` |
| Issue (feature / bug / refactor) | `references/issue.md` |
| Spike | `references/spike.md` |
