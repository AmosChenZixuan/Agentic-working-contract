---
name: agent-code
description: Use this skill to execute a single plan slice from agent-plan, or for any self-contained coding task with a clear requirement. Triggers on "implement X", "fix Y", "execute slice N", "code this up", or any handoff from agent-plan. Replaces agent-coding-contract.
---

# agent-code

You are in phase 3 of 3. You own one slice from start to passing tests. Work through every phase in order — each gates the next.

**Chain:** `agent-scope` → `agent-plan` → `agent-code`

---

## Phase 1: Read the slice

If handed a plan file, read the specific slice you're executing. Extract:
- The goal and acceptance criteria
- Files to touch
- The closing boundary test to write
- Dependencies (confirm prior slices are complete)

If no plan exists (direct invocation), derive your own scope: one coherent behavior change, one closing boundary test, nothing more.

**Ask any clarifying questions now — before writing a single line of code.** Use `interrogate-me` if available.

---

## Phase 2: Inspect

Read the files you'll change — not just the target, but its callers, imports, and related tests. Understand:
- Existing patterns: naming, error handling, test structure
- What must stay the same
- Where the change fits without breaking neighbors

**Deliverable:** A 2–3 sentence internal summary of what exists today and where this slice fits.

---

## Phase 3: Write the closing boundary test

Write the test before any implementation code. It must:
- Fail now (run it — confirm the failure is for the right reason, not a syntax error)
- Pass only when the slice's goal is fully met
- Cover the acceptance criteria from the slice — not more, not less
- Match the project's existing test style exactly

This test is your definition of done. When it's green, the slice is complete.

---

## Phase 4: Implement

Write the minimum code to make the closing boundary test pass. Add comments where logic isn't self-evident — explain *why*, not *what*

- Follow the patterns observed in Phase 2 — be a consistent contributor
- No error handling for impossible states, no abstractions for hypothetical reuse, no features not in the slice
- Run the closing boundary test after each meaningful change

---

## Phase 5: Verify

Run the full test suite (or the relevant subset for large repos).

- All tests pass, including pre-existing ones
- No new lint, type, or format errors
- If something unexpected fails, investigate — don't paper over it

Read the output. Do not claim success until you have.

---

## Phase 6: Scope check

Read the diff. Ask: did you build what the slice asked for, and only that?

- Flag anything you added that wasn't in the slice
- Remove speculative additions, extra abstractions, or "while I'm here" changes
- If you found a real problem outside the slice, note it for the next slice — don't fix it now

Then ask: can any of this be simpler?
- Is there an existing utility that already does what you just wrote?
- Are there magic numbers, repeated patterns, or inlined logic that should be named?

Re-run the closing boundary test after any changes.

---

## Phase 7: Report

```
**Slice:** [name or description]
**What changed:** [1–2 sentences]
**Test coverage:** [what the closing boundary test verifies]
**Out-of-scope observations:** [anything noticed but deliberately not touched]
```

**Suggested commit message** — run `git log --oneline -3`, match the project's format exactly.

**Next:** If there are remaining slices in the plan, name the next one. Otherwise: "All slices complete — ready to ship."

**Note:** Always ask for using subagents to work on independent slices in parallel. 