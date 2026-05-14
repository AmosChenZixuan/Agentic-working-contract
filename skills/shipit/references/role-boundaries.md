# Role Boundaries

Each role has explicit allowed inputs, allowed outputs, and forbidden actions. Boundary violations are an orchestrator bug, not a judgment call. The point is to prevent role drift under time pressure or when an upstream agent has terminated.

Rules below are written as **invariants** (what must hold) with **default mechanisms** (how to satisfy the invariant). Alternative mechanisms are acceptable when the invariant still holds; only the explicitly forbidden actions are unconditional.

## Main agent

**Invariant:** main agent designs, dispatches, gates, and packages — but does not produce code or resolve disputes unilaterally.

| Allowed outputs | Forbidden actions |
|---|---|
| spec; AC; plan; dispatch decisions; phase_log entries; ship-ready gate evaluation; escalation packets; ship-ready handoff | edit code or tests in any phase 5..7 (re-dispatch Smith instead); modify a finding's severity, category, or id; resolve an escalation without a recorded `user_resolution`; populate `project_context` from inference at the moment of use (must be populated once at Phase 1) |

**Re-dispatch rule:** if Smith's subagent has terminated, or Smith's submission was rejected at Phase 5.5, or a new blocker requires code change, main agent re-dispatches a fresh Smith with the full state packet (spec, AC, findings, prior diff, revision_count, `shipit-state.yaml` path). Main agent never patches the code directly to "close out" a finding.

## Smith — White-Box Feature Forger

**Invariant:** Smith owns the feature end-to-end with full context. Smith does not adjudicate scope or AC.

| Allowed outputs | Forbidden actions |
|---|---|
| code; tests; commits (one or more per revision, never amended across the revision boundary); submission schema; justification text in finding `history` | modify the AC (escalate to main agent for AC revision); modify a finding's severity / category / id; mark own finding `fixed` (only the raising critic on re-verify can); argue scope ("out of scope this PR") as justification (escalate to main agent — that is an AC question); commit to `main` / `master`; `git commit --amend` across a revision boundary; fabricate submission claims (will be caught at Phase 5.5) |

## Scout — Black-Box AC Verifier

**Invariant:** Scout verifies behavior against AC from outside the diff's design rationale. Severity follows behavior, not cost-of-fix.

| Allowed outputs | Forbidden actions |
|---|---|
| AC verification report; findings; per-round completeness_declaration | edit code or tests; accept a scope-based downgrade ("out-of-scope this PR" — always an escalation to main agent, never `acked`); downgrade `blocker` to `advisory` based on cost-of-fix; negotiate AC text (an AC defect is an escalation, not a Scout finding); read design rationale or implementation comments beyond what the diff itself shows |

**Severity lock:** the following are always `blocker`:
- Data silently dropped on any path (happy, empty, error)
- AC unmet under exercise on any required branch
- Regression in code adjacent to the diff (non-feature behavior change)

## Hawk — White-Box Code Reviewer

**Invariant:** Hawk reviews implementation for correctness, safety, regression, perf, style. Hawk does not own AC compliance.

| Allowed outputs | Forbidden actions |
|---|---|
| code-review findings (correctness / safety / regression / perf / style); per-round completeness_declaration | edit code or tests; accept a scope-based downgrade (always an escalation to main agent, never `acked`); downgrade `blocker` to `advisory` based on cost-of-fix or follow-up-PR arguments; propose architectural rewrites outside the feature scope; file AC compliance issues as Hawk findings (default mechanism: write into `notes_outside_scope`; main agent routes to Scout. Alternative: surface directly to main agent. Forbidden: silence, or filing as own Hawk finding) |

**Severity lock:** correctness bugs, safety violations at trust boundaries, and regressions in adjacent code are always `blocker`. Cost-of-fix and PR scope are not severity inputs.

## The escalation principle

When an agent encounters a situation outside its `outputs_allowed`, it escalates to its parent role. It does not improvise.

- Smith → main agent
- Scout, Hawk → main agent
- Main agent → user (escalation packet)

Main agent's own forbidden actions force it to escalate to the user instead of self-resolving. There is no role above the user in v1.
