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

Each item must be observable and testable. Each gets a stable id (`AC1`, `AC2`, …). Each item also declares which **branches** it requires (happy / empty / error / etc.) and which **emit↔consume pairs** it spans. These declarations feed Smith's `ac_branches_tested` and Scout's `emit_consume_chains_walked`.

```yaml
- id: AC1
  behavior: <concrete observable, e.g., "POST /login with valid credentials returns 200 with a session token in the response body">
  branches_required: [happy, empty, error]      # which branches MUST be tested
                                                # `happy` = nominal path
                                                # `empty` = null / empty input / no-data path
                                                # `error` = failure path (invalid input, downstream failure, etc.)
                                                # Use only the branches that actually exist for this AC; do not pad.
  emit_consume_pairs:                           # any cross-surface data flow this AC requires
    - emit:    <where the value is produced, e.g., "server: persist_chat_turn writes citations.context[]">
      consume: <where the value is read,     e.g., "client: ChatPanel renders context items">
  testable_via: <test surface, e.g., "pytest backend integration; playwright client smoke">
```

Repeat for AC2, AC3, …

**Hard rule:** if a behavior has a meaningful negative path (empty input, missing data, downstream failure), `branches_required` MUST include `empty` and/or `error`. "Works correctly under good inputs" is not a complete AC.

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

## Hard gates before Smith dispatch

- ≥1 AC item that is observable and testable
- No AC reads "works correctly" or equivalent vague phrasing
- Every AC has a non-empty `branches_required` list, and AC items with meaningful negative paths include `empty` and/or `error`
- Every AC that spans surfaces has `emit_consume_pairs` populated
- Token budget estimated and within bound
- All open questions either resolved or explicitly deferred with rationale
- Design decisions log non-empty (if no decisions were needed, write "trivial — no decisions" explicitly, don't leave the section blank)
