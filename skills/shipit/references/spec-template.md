# Spec + AC Template

Main agent writes the spec during Phase 3 of shipit. Save to `docs/specs/YYYY-MM-DD-<slug>.md`.

## Template

```markdown
# <Feature name>

## Problem
<What this solves, for whom, why now. 2–4 sentences.>

## Approach
<Chosen design. Name affected files, interfaces, patterns. Concrete enough for A to implement without guessing intent.>

## Scope

### In scope
- <concrete deliverable>
- <concrete deliverable>

### Out of scope
- <explicit exclusion to prevent scope creep>

## Acceptance criteria

Each item must be observable and testable. Each gets a stable id (`AC1`, `AC2`, …).

- **AC1:** <concrete behavior, e.g., "POST /login with valid credentials returns 200 with a session token in the response body">
- **AC2:** <concrete behavior>
- **AC3:** <concrete behavior>

## Design decisions log

Decisions main agent already made during clarification. These are constraints A must respect, not suggestions A can revisit.

- <decision: chose X over Y because Z>
- <decision: rejected approach W because V>
- <decision: existing pattern P will be reused>

This log survives compaction (if compaction ever occurs). It also seeds the escalation context if A challenges a decision.

## Open questions

- <any unresolved item — must be resolved or explicitly deferred before A dispatch>

## Token budget estimate

- Spec + AC payload: ~<N> tokens
- Code reads A will need (files A must read fully): ~<N> tokens
- Total A initial load: ~<N> tokens

**Gate:** total ≤ 60k. If over, split feature into multiple PRs upstream — do not proceed to Phase 5.

## Heuristic gate (fallback when token estimate is imprecise)

If any of the following are likely true, split:
- Diff > 800 LOC
- Files touched > 8
- More than one independently-shippable behavior change

## PR metadata (optional, fill in at ship-ready time)

- Suggested commit prefix: <feat | fix | refactor | …>
- Suggested PR title: <short>
- Affected reviewers / CODEOWNERS: <if known>
```

## Hard gates before A dispatch

- ≥1 AC item that is observable and testable
- No AC reads "works correctly" or equivalent vague phrasing
- Token budget estimated and within bound
- All open questions either resolved or explicitly deferred with rationale
- Design decisions log non-empty (if no decisions were needed, write "trivial — no decisions" explicitly, don't leave the section blank)
