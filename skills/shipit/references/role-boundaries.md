# Role Boundaries

Each role has explicit allowed inputs, allowed outputs, and forbidden actions. Boundary violations are an orchestrator bug, not a judgment call. The point is to prevent role drift under time pressure or when an upstream agent has terminated.

Rules below are written as **invariants** (what must hold) with **default mechanisms** (how to satisfy the invariant). Alternative mechanisms are acceptable when the invariant still holds; only the explicitly forbidden actions are unconditional.

## Main agent

**Invariant:** main agent designs, dispatches, gates, and packages — but does not produce code or resolve disputes unilaterally.

| Allowed outputs | Forbidden actions |
|---|---|
| spec; AC; plan; dispatch decisions; phase_log entries; ship-ready gate evaluation + `gate_evidence`; escalation packets; ship-ready handoff | edit code or tests in any phase 5..7 (re-dispatch Smith instead); modify a finding's severity, category, or id; resolve an escalation without a recorded `user_resolution`; populate `project_context` from inference at the moment of use (must be populated once at Phase 1); perform git history surgery (cherry-pick / amend / rebase / reset) to recover a git-surface anomaly (escalate instead — see below) |

**No triviality exception.** Main agent editing code or tests in phases 5–7 is an invariant violation **regardless of how small the change is**. There is no "maintainer patch", "trivial one-liner", or "just unblock it" carve-out — none exists anywhere in this skill; inventing one at runtime is itself the violation. A one-line bug in Smith's output is a finding routed to Smith, not a main-agent edit. The cost argument ("re-dispatch is slow") is answered by incremental re-dispatch below, not by a boundary exception.

Re-dispatch is incremental and goes to the persisted Smith (see § Subagent continuity), never to a main-agent patch: the narrow finding only, prior commits intact. A one-line fix is ~1 minute — that cheap path is why no triviality exception is needed. Main agent never patches code to close a finding.

## Subagent continuity (Smith, Scout, Hawk)

Every shipit subagent is spawned **once** and persists for the whole run; its handle is recorded in `counters.agent_ids.<role>`. Every later contact — a Smith revision, a Scout/Hawk re-verification of a new revision — is a `SendMessage` to that handle carrying only the delta (the finding, or `re-verify HEAD <sha>`), never a fresh `Agent` spawn. The subagent already holds full context.

A fresh spawn is permitted **only when the handle is structurally unrecoverable**: the subagent terminated/errored, or the main session was compacted/resumed (the state file survives a resume; an in-memory handle does not). A fresh spawn while the handle is alive is a contract violation, not an optimization.

A cold-respawned subagent has no run context and **must reconstruct from on-disk artifacts before producing output** (acting on the delta alone is the characteristic cold failure):

- **Smith** — prior commits (`base_sha..HEAD`) + current diff; last `submission.<N>.yaml` + `verification_report.<N>.yaml`; every finding `history`; the spec's `emit_consume_pairs`, traced end-to-end (emit→transport→persist→consume) before touching any producer/consumer.
- **Scout / Hawk** — own prior-round file; findings it raised + their `history`; the current diff.

**Context-pressure guard (all roles).** A persisted subagent nearing its own context limit hands back (`status: incomplete` / spiral flag) — never silently self-summarizes; silent internal compaction reintroduces the context loss persistence exists to prevent.

### git-surface anomaly (no history-surgery recovery)

A **git-surface anomaly** is any disagreement between the probed `git_anchor` and live git: current branch ≠ `git_anchor.feature_branch`; `base_ref` advanced past `git_anchor.base_sha` (a feature commit landed on base/`master`); a missing or duplicated `Shipit-Revision` trailer; HEAD on the wrong worktree. These are detected by the Phase 8 `PG` probe and may surface mid-run.

**On any git-surface anomaly: STOP and escalate to the user with an escalation packet.** This is the same shape as the no-triviality exception: the contract has no sanctioned self-recovery, so inventing one *is* the violation.

**Forbidden recovery improvisations:** `git cherry-pick` of the misrouted commit (produces duplicate revision commits, destroying the Invariant 2 anchor); `git commit --amend` / `rebase` / `reset` / `filter-branch` to tidy history; re-committing the same change in a second context to match the anchor.

**Sanctioned path:** escalate. If the user authorizes a redo, re-dispatch Smith on the pinned worktree surface (`git -C "$git_anchor.worktree_path"`) from `git_anchor.base_sha`, prior misrouted commits abandoned, not transplanted. Git history surgery to rescue a run is a user-authorized operation, never a main-agent improvisation.

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
