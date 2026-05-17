---
name: shipit
description: Use when delivering a single PR-scoped feature end-to-end with multi-critic review loops. Triggers on "/shipit", "ship this feature", or any request for full-cycle feature delivery from design through ship-ready PR with structured critique.
metadata:
  version: 0.1.0
---

# shipit

Orchestrator. Main agent owns design, AC, planning, the ship gate, and arbitration. It spawns three named subagents per feature, each **once**, persisting for the whole run:

- **Smith** — white-box implementer: design → tests → code → refactor. Full feature context, no compaction.
- **Scout** — black-box AC verifier: exercises the feature from outside (playwright MCP for web-ui; curl / shell / test-runner otherwise). Never reads design rationale.
- **Hawk** — white-box reviewer: correctness, safety, regression, perf, style.

Scout and Hawk run in parallel after Smith's submission is verified, feeding lean findings back to Smith in a bounded loop. Output: a ship-ready **draft** PR — never auto-merged.

## Principles

- **Context-boundary.** Smith holds full feature context end-to-end and persists for the run (contact via SendMessage; cold respawn only if the handle is dead). No mid-flight compaction.
- **Evidence boundary (hard).** A "done" without an attached test tail is not done. A submission lacking `verification_evidence` / `tests_passed: true` is invalid at the boundary — not a thing to verify later.
- **Branch isolation (hard).** Never main/master. Verify the branch before the PR. Never auto-merge — output a draft PR.
- **Structured critique.** Scout (black-box AC) ∥ Hawk (white-box code) after submit. Lean findings, no prose. Pushback ≤3 per finding id.
- **Roles don't drift.** Main designs / dispatches / gates and never edits code in the loop. Critics review, never edit. Escalate, don't improvise. The user is sole arbiter of escalations.
- **State on disk.** spec + findings + a small status file under `docs/specs/<date>-<slug>/`. Survives compaction and resume.
- **Skill prefer / fallback.** Prefer an installed skill per step; else degrade inline. Never auto-install. If a step degrades, tell the user once.

## Single PR scope

Smith's initial context ≤ ~60k tokens. If the diff likely exceeds ~800 LOC or touches >8 files, split the feature upstream at step 3 and stop.

## Skill preference

| Step | Prefer | Else |
|---|---|---|
| Clarify | `superpowers:brainstorming` / `grill-me` | inline Q&A |
| Plan | `superpowers:writing-plans` | inline slice list |
| Smith impl | `superpowers:test-driven-development` + `superpowers:verification-before-completion` | inline TDD + verify |
| Workspace | `superpowers:using-git-worktrees` | `git worktree add` / `git checkout -b` |
| Hawk review | platform `/review` | `c-simplify` / inline |
| Cleanup | `neat-freak` | inline doc/memory update |
| Commit msg | `caveman:caveman-commit` | inline Conventional Commits |

## Flow

State lives in `docs/specs/<date>-<slug>/status.yaml`:

```yaml
slug:             <feature-slug>
branch:           <feature branch>
worktree:         <abs path | null>
step:             1            # 1..8
smith_dispatches: 0            # cap 6
findings:         []           # [{id, severity, status}]
pr_url:           null
```

**1. Setup.** Determine workspace mode — honor a stated user choice; else ask `[branch / worktree / abort]`; never a silent default. On main/master, refuse and prompt. `branch` → `git checkout -b <slug>` from the repo's main ref. `worktree` → `superpowers:using-git-worktrees` if available, else `git worktree add`. Create `docs/specs/<date>-<slug>/`, write `status.yaml`, `cd` into the workspace.

**2. Clarify + spec.** Clarify intent (skill or inline) until problem / success / out-of-scope / constraints are answered. Write `spec.md`: problem, approach, scope (in / out), and the acceptance criteria. Each AC is observable and testable, has a stable id (`AC1`…), and declares `branches_required` (`happy`, plus `empty`/`error` where a negative path exists). At least one AC; no AC may read "works correctly".

**3. Plan + size.** Plan the work (skill or inline slice list). Estimate Smith's initial load and the diff size. If >~60k tokens, >~800 LOC, or >8 files, split into multiple PRs and stop here.

**4. Dispatch Smith.** Spawn Smith once (`subagents/smith.md`) with spec, AC, plan, and the `status.yaml` path. Bump `smith_dispatches`. Smith: write closing tests per AC branch → minimal implementation → sweep renamed/removed symbols → run `$test_command` → commit (subject per repo convention; the revision's last commit trailers `Shipit-Revision: <N>`) → submit the schema with `verification_evidence`.

**5. Verify submission.** Admission: `verification_evidence` absent, or `tests_passed != true`, or `status: incomplete` → invalid at the boundary; re-dispatch Smith incrementally with the gap. If admitted, re-run `$test_command` **once** independently:
- Re-run clean and consistent with the attached tail → accept; persist the raw submission to `submission.<N>.yaml`; re-check the step-3 size heuristic against the real diff (escalate if exceeded).
- Re-run contradicts the attached evidence → fabrication; escalate to the user (do not silently re-dispatch).

**6. Scout ∥ Hawk.** With Smith idle (not terminated), spawn Scout (`subagents/scout.md`) and Hawk (`subagents/hawk.md`) once each on the accepted commit. Pass each the verified diff and the `status.yaml` path; pass Scout the feature type; do not pass Scout design rationale. Record the reviewed SHA.

**7. Critique loop.** Collect findings. SendMessage blockers to the persisted Smith (cold respawn only if its handle is dead). Smith revises (≥1 new commit per revision). SendMessage Scout and Hawk to re-verify the new HEAD; update the reviewed SHA. Pushback ≤3 per finding id; then that finding escalates. Escalate also on a Scout↔Hawk conflict on the same code, or 2 consecutive revisions with no blocker-count decrease. On escalation, stop and ask the user with the finding + both sides + options (side with critic / override / revise AC / revise spec / accept-escalated / abort); record the decision on the finding.

**8. Ship gate → handoff → cleanup → reflection.**

Gate — passes only if every item holds, each an independent artifact, not main's assertion. Missing any → not ship-ready, loop back; main may not substitute its own judgment for a missing artifact.

- Smith's last `verification_evidence` was re-run clean by main at step 5 (record exists, exit 0).
- Scout's last report: every AC verified pass.
- Hawk's last report: zero `open` blocker.
- `git rev-parse --abbrev-ref HEAD` ∉ {main, master}.
- Every `escalated` finding has a recorded user decision.

On pass: `git push -u <remote> <branch>`, then `gh pr create --draft` (title per repo convention; body = Scout AC report + Hawk summary + finding log). Set `pr_url`.

Handoff: output the PR url, Scout report, Hawk summary, finding log, escalations, and branch / worktree path.

Cleanup (not gated): default on — `neat-freak` if available, else an inline doc/memory update. The user may skip with one word; no waiver file.

Reflection (not gated): one short main-written retro (what was done, honest misses), surfaced to the user. A separate `session-reflection.md` only if the user asks.

Forbidden in step 8: `gh pr merge`; un-drafting the PR; `git push --force` (except executing an explicit user-authorized escalation).

## Lean finding

Every critic finding uses this shape. No prose reviews.

```yaml
id:        <critic>-<short-slug>   # stable; reuse the exact id on re-raise
raised_by: scout | hawk
severity:  blocker | advisory
issue:     <=2 sentences
fix:       <=2 sentences           # optional for advisory
status:    open | fixed | acked | escalated
pushback:  0                       # increment on re-raise; 3 → escalate
```

`blocker` must be fixed, critic-accepted, or escalated→user-resolved before ship. `advisory`: Smith acks and decides, no loop. Severity follows behavior, not cost-of-fix or PR scope — silently dropped data, an AC unmet on a required branch, an adjacent regression, and a safety violation at a trust boundary are always `blocker`. `fixed`/`acked` are terminal; `escalated` is terminal only after the user decides.

## Hard rules

The only unconditional rules.

- Never auto-merge; output a draft PR.
- Never work on main/master.
- Scout + Hawk run on every feature; no directive (including a reflection request) waives them.
- A submission without attached falsifiable test evidence is invalid — not verify-later.
- Main never edits code/tests in the critique loop; re-dispatch Smith incrementally.
- The user is sole arbiter of escalations; main packages context, does not decide.
