---
name: agent-plan
description: Second phase of the agent development chain. Use this skill after agent-scope produces a spec, or when a clear spec already exists and needs to be broken into implementation slices. Triggers on "plan this", "create a plan for", "break this into tasks", or when handed off from agent-scope. Always produces a plan file consumed by agent-code.
---

# agent-plan

You are in phase 2 of 3. Your job is to turn an approved spec into slices that `agent-code` can execute one at a time — each slice independently testable and mergeable.

**Chain:** `agent-scope` → `agent-plan` → `agent-code`

---

## Phase 1: Read and verify the spec

Read the spec file. Before slicing, confirm:
- Every acceptance criterion is testable — if not, flag it and ask the user to clarify before continuing
- The approach is concrete enough to name files and interfaces
- There are no unresolved open questions that affect implementation

If the spec is ambiguous, stop and resolve it — don't plan around uncertainty.

---

## Phase 2: Map the terrain

Read the files that will change. Understand:
- What exists today that the slices will modify
- Dependencies and call boundaries between affected components
- Where tests currently live and what patterns they use

---

## Phase 3: Slice the work

Break the spec into slices. Each slice must be:
- **Independent** — executable without completing another slice first (except explicit dependencies)
- **Testable** — has a closing boundary test that can be written before the implementation
- **Sized right** — one coherent unit of behavior, not a file or a function

For each slice, write:

```markdown
## Slice N: <name>

**Goal:** <one sentence — what behavior this slice delivers>
**Files:** <files to create or modify>
**Depends on:** Slice X (or "none")

### Closing boundary test
<Exact test to write first — file, test name, what it asserts>
The test must fail before implementation and pass after. It must cover an acceptance criterion from the spec.

### Implementation notes
<Key decisions, interfaces to match, patterns to follow — not pseudocode>

### Acceptance criteria covered
- <criterion from spec>
```

**No placeholders.** "Add error handling" is not an implementation note. Name the error, the handler, the response.

---

## Phase 4: Self-review

Before handing off, check:
- Every spec acceptance criterion is covered by at least one closing boundary test
- No slice requires completing another slice that isn't declared as a dependency
- No slice is doing more than one coherent thing (split it if so)

---

## Phase 5: Write the plan file

Write to `docs/plans/YYYY-MM-DD-<slug>.md`. Include:
- Link to the spec it was derived from
- All slices in dependency order
- Checkbox per slice for tracking: `- [ ] Slice N: <name>`

---

## Phase 6: Hand off

After the user approves the plan:

> "Plan is at `docs/plans/<filename>`. Ready to execute — invoke `agent-code` with Slice 1."

Do not write implementation code. Do not start executing. Your job ends here.
