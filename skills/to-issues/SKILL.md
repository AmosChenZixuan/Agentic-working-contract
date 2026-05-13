---
name: to-issues
description: Create agent-ready GitHub issues from specs, feature docs, conversations, or ad-hoc findings. Handles both spec decomposition (epic + layered sub-issues) and standalone issues (single bug, refactor, feature). Use when the user invokes /to-issues or /to-issue, says "break this into issues", "create issues from this spec", "file an issue for this", "convert to backlog", or when conversation surfaces a bug, design gap, or deferred work worth capturing. This is the counterpart to /clear-issues — it produces the backlog that /clear-issues consumes.
---

# to-issues

Create agent-ready GitHub issues — structured so `/clear-issues` can grill, dispatch, and implement them with minimal back-and-forth.

Two modes:
- **Decomposition** — a spec or feature doc becomes an epic with layered, dependency-ordered sub-issues
- **Standalone** — a single finding (bug, refactor point, feature idea) from conversation becomes one issue, no epic

## What makes an issue "agent-ready"

The downstream consumer is `/clear-issues`, which runs a grill phase extracting:
- **Root cause / goal** — what exactly needs to happen
- **Files to touch** — which modules, components, or files are in scope
- **Test strategy** — how to verify the work is done
- **Platform** — backend / web / both
- **Scope boundary** — what is explicitly NOT in scope

Every issue this skill produces must make those five things answerable from the issue body alone, without needing to read the full spec or ask clarifying questions.

---

## Mode detection

Determine the mode from context:

- **Decomposition mode** when: user points to a spec/doc file, or the work clearly involves multiple components, layers, or both backend and frontend.
- **Standalone mode** when: user describes a single finding (bug, refactor, feature idea), or conversation surfaced one specific issue worth capturing.

If ambiguous, ask: "Is this one issue or should I break it into a backlog?"

### Standalone mode

Skip Steps 2-3. Go directly to:

1. Run Step 1 (gather context) — repo info, labels, commit style, relevant codebase patterns
2. Detect issue type from context: feature, bug, or refactor
3. Load the matching template from `references/` — omit `Parent:`, `Layer:`, `Depends on:` fields (they say "or omit if standalone")
4. Fill the template from conversation context. If a section can't be filled, ask the user rather than guessing.
5. Submit via `gh issue create` and return the URL

The rest of this document covers **decomposition mode**.

---

## Step 1: Gather context

Read the spec file the user points to. Then gather codebase context:

1. **Repo info** — `gh repo view --json nameWithOwner` and `gh label list` for available labels
2. **Commit style** — `gh issue list --limit 10` and `git log --oneline -10` to match title prefix conventions
3. **Existing patterns** — scan the codebase areas the spec touches to understand current file structure, naming, types, API patterns, and test patterns. This grounds the issues in reality rather than speculation.

Do not skip step 3. Issues that name real files, real types, and real patterns are dramatically faster for agents to execute than issues that say "follow existing patterns."

---

## Step 2: Decompose — three passes

### Pass 1: Find the seams

Identify boundaries where work can happen independently.

- **Backend vs frontend** is the strongest seam — connected by an API contract. Once the contract is agreed, both sides build independently.
- **Within frontend:** separate "renders without real data" (panel shells, layout) from "needs live data" (lists with polling, forms that POST).
- **Within backend:** separate "data model + API surface" from "business logic / pipeline."
- **For bug fixes:** the seam is usually diagnosis vs fix. Multiple independent root causes = multiple issues.
- **For refactors:** the seam is mechanical (renames, moves) vs behavioral (new abstractions, interface changes).

### Pass 2: Smallest useful unit

Each issue must produce something testable in isolation.

- **Merge** things that aren't useful alone (DB schema without endpoints is useless → combine them).
- **Split** things that are independently useful (a list component works without its context menu).
- **For refactors:** the constraint is "reviewable in one sitting" rather than "demo-able."
- **Never split by file** — "one issue for types.ts, one for api.ts" creates issues nobody can work on independently.
- **Never bundle fix + cleanup** — different risk profiles, different PRs.

### Pass 3: Dependency graph

Trace data flow backwards from leaf features to find the natural layers.

- Items at the same layer with no shared dependency are **parallelizable**.
- Each issue must be **mergeable to master independently** without breaking anything.
- Label each issue with its layer number and explicit `depends-on` references.

---

## Step 3: Propose before creating

Present the decomposition to the user as:

1. A numbered list with title, platform label, and 1-line summary for each issue
2. An ASCII dependency diagram showing the layers
3. A note on what's parallelizable

Ask for confirmation before creating any issues. The user may want to merge, split, reorder, or drop items.

---

## Step 4: Create the issues

After user confirms, create issues using `gh issue create`.

### Templates

Load the appropriate template from `references/` for each issue:

- `references/epic.md` — the parent tracking issue (always created first)
- `references/feature.md` — sub-issues for new functionality
- `references/bug.md` — sub-issues for bug fixes
- `references/refactor.md` — sub-issues for refactoring work

Fill every field. The **structural metadata header** (`Parent:`, `Layer:`, `Depends on:`) is the contract with `/clear-issues` — it must be present on every sub-issue, exactly as formatted in the templates.

### Writing the Scope / Fix approach / Target state section

This is the most important section regardless of template. It should contain enough detail that an agent reading only this issue (not the spec) can implement the work. Include:
- Schema definitions (SQL, TypeScript interfaces) if applicable
- API endpoint signatures if applicable
- Component props and structure if applicable
- Which existing patterns to follow, with file references

### Writing Acceptance criteria

Must be objectively verifiable — "works correctly" is not a criterion, "GET /api/artifacts returns 200 with list of artifacts" is.

### Labels and title format

- Match the repo's existing label set (checked in Step 1)
- Match the repo's title/commit prefix convention (checked in Step 1)
- Apply platform labels (`backend`, `web`, etc.) when available

### After creation

Update the epic issue body with actual issue numbers (replace placeholder `#NN` references).

---

## Adapting to work type

The three-pass decomposition works for features, bugs, and refactors, but emphasis shifts:

| Work type | Pass 1 emphasis | Pass 2 constraint | Pass 3 note |
|-----------|----------------|-------------------|-------------|
| Feature | Backend/frontend seam | Demo-able | Layers often deep (3+) |
| Bug fix | Independent root causes | Atomic (1 cause = 1 issue) | Usually flat (1 layer) |
| Refactor | Mechanical vs behavioral | Reviewable in one sitting | Chains more than fans |
