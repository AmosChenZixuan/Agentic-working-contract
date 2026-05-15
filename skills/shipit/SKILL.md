---
name: shipit
description: Use when delivering a single PR-scoped feature end-to-end with multi-critic review loops. Triggers on "/shipit", "ship this feature", or any request for full-cycle feature delivery from design through ship-ready PR with structured critique.
metadata:
  version: 0.0.5
---

# shipit

Orchestrator skill. Main agent owns design, AC, planning, gate evaluation, and arbitration. Spawns three named subagents per feature:

- **Smith** — white-box implementer; forges the feature end-to-end (design → tests → code → refactor); keeps full feature context with no mid-flight compaction.
- **Scout** — black-box AC verifier; reconnoiters from outside; exercises the feature via the appropriate tool (playwright MCP for web UI, curl / shell / test-runner otherwise) without reading design rationale.
- **Hawk** — white-box code reviewer; sharp eyes for correctness, safety, regression, perf, style.

Scout and Hawk run in parallel after Smith submits and feed structured findings back to Smith in a bounded critique loop. Output is a ship-ready draft PR — never auto-merged.

## Core principles

- **Context-boundary over task-boundary.** Smith holds full feature context end-to-end. No mid-flight compaction. Eliminates inter-agent re-derivation cost.
- **Single PR scope.** Smith's initial loaded context ≤60k tokens. If diff likely >800 LOC or touches >8 files, split feature upstream during planning.
- **Parallel critics, structured findings only.** Scout + Hawk run in parallel after Smith's first submission. All findings use the schema in `references/finding-schema.md`. No prose reviews.
- **Per-finding pushback budget = 3.** Critic re-raises same finding id at most 3 times. Then that finding escalates. Loop continues on other findings until no blockers remain.
- **Spiral detector.** Two consecutive Smith revisions without blocker-count strictly decreasing → escalate.
- **Never auto-ship.** End state is a draft PR. User merges manually.
- **Never on main.** Smith refuses to work on `main` / `master`. Worktree preferred, feature branch fallback.
- **State lives in a file, not in conversation.** All run state — phase log, project context, counters, findings index, anchored review ref — lives in `shipit-state.yaml` per `references/state-file.md`. Conversation context is too fragile to back a state machine.

## Reference files

| File | What's in it |
|---|---|
| `references/state-file.md`              | `shipit-state.yaml` schema; SSOT for run state |
| `references/role-boundaries.md`         | Role contracts (main agent, Smith, Scout, Hawk) |
| `references/completeness-declaration.md`| Per-role completeness schemas |
| `references/audit-invariants.md`        | Finding-history, git-history, phase-log invariants |
| `references/finding-schema.md`          | Structured finding fields, severity, id discipline |
| `references/escalation-packet.md`       | Escalation triggers and format |
| `references/spec-template.md`           | Spec + AC template |
| `references/skill-fallback-matrix.md`   | Per-phase fallback chain and install commands |
| `subagents/smith.md`                    | Smith's prompt |
| `subagents/scout.md`                    | Scout's prompt |
| `subagents/hawk.md`                     | Hawk's prompt |

## Phases

shipit is a state machine. Each phase has entry conditions (must hold to start) and exit conditions (must hold to leave). Every phase appends an entry to `shipit-state.yaml → phase_log`; the ship-ready gate consumes the log directly. Phases cannot be skipped, reordered, or marked complete without their exit conditions actually holding.

### 0a. Branch / worktree check

- If on `main` / `master`: block, prompt user `[worktree / branch / abort]`.
- If `superpowers:using-git-worktrees` available, defer to it; else `git worktree add` inline.
- Create `docs/specs/YYYY-MM-DD-<slug>/` and initialize `shipit-state.yaml` with the first phase_log entry. Record the **absolute** worktree path.

**Exit:** `shipit-state.yaml` exists; `phase_log[0]` recorded; current `pwd` is the worktree.

### 0b. Skill / MCP probe + tooling probe

Probe skill availability per `references/skill-fallback-matrix.md`. **Also probe**:

```bash
gh auth status                             # for Phase 9 PR creation
git remote -v                              # default_remote
```

Record degraded tiers; prompt user once whether to install or proceed degraded.

**Exit:** per-phase tier table recorded as artifact; `gh_authenticated` + `default_remote` known.

### 1. Background gather + project_context

Read repo: `git log --oneline -20`, existing patterns, related code, prior specs.

**Populate `shipit-state.yaml → project_context` in full.** All later phases and subagents read from here; none re-derive. Detect and record:

- `base_ref` — what the feature branch branched from (e.g. `origin/main`)
- `test_command` — sniff `package.json` scripts, `Makefile`, `pyproject.toml`, `pytest.ini`, `cargo.toml`, etc.
- `lint_command` — same approach
- `commit_convention` — infer from `git log --oneline -20` + optional `CONTRIBUTING.md` / `.gitmessage`
- `pr_title_convention` — infer from `gh pr list --state merged --limit 10` (skip if `gh_authenticated == false`)
- `feature_type` — web-ui / http-api / cli / library / mixed

A missing value here is a Phase 1 defect — return to Phase 1, do not work around at point of use. See `references/state-file.md`.

**Exit:** `project_context` populated; all fields non-null (or explicitly recorded as `unavailable` with reason).

### 2. Clarify intent

Prefer `superpowers:brainstorming`. Local fallback `grill-me`. Degrade: one question per turn until problem / success / out-of-scope / constraints are answered.

### 3. Write spec + AC

Use `references/spec-template.md`. Write to `docs/specs/YYYY-MM-DD-<slug>/spec.md`. Each AC declares `branches_required` and `emit_consume_pairs`. Hard gate: at least one observable, testable AC; AC items with meaningful negative paths must declare `empty` and/or `error` in `branches_required`.

### 4. Plan

Prefer `superpowers:writing-plans`. Estimate Smith's initial token load. If >60k or diff/file heuristic exceeded, split feature into multiple PRs and stop.

### 5. Dispatch Smith

Spawn subagent with `subagents/smith.md`. Pass: spec, AC, plan, design decisions log, and the path to `shipit-state.yaml` (Smith reads `project_context.worktree_path`, `test_command`, etc. from there). Bump `counters.smith_dispatch_count`.

**Exit:** Smith returns a submission schema. Submission is an **assertion** at this point — Phase 5.5 verifies it before it counts as fact.

### 5.5. Submission verification (mechanical, non-skippable)

Smith's submission is a self-report. Subagent output cannot be trusted on its face — Smith may hallucinate code that was never written. Main agent verifies mechanically before Phase 6 dispatch.

From `project_context.worktree_path`:

```bash
git -C "$worktree_path" status --porcelain
git -C "$worktree_path" diff --name-only "$base_ref"...HEAD     # files actually changed
git -C "$worktree_path" log --oneline "$base_ref"..HEAD         # commits actually made
$test_command                                                   # actual test count + pass/fail
```

Cross-check against Smith's submission per the table in `references/state-file.md`. Any mismatch → submission is `rejected_unverified`. Main agent's only allowed action is **re-dispatch Smith** with the failure report. Main agent does NOT patch the gap itself (role contract — see `references/role-boundaries.md`).

Write `verification_report.<N>.yaml` (verdict `accept` | `reject` + evidence) per dispatch. `verification_failure_streak` is derived at check time as the count of trailing `reject` verdicts. If streak ≥ 2 or `smith_dispatch_count` ≥ 6, escalate to user.

**Exit:** all claims verified; re-check Phase 4's size heuristic against actual diff (escalate if exceeded); `verification_report` artifact written.

### 6. Dispatch Scout + Hawk in parallel

**Entry:** Phase 5.5 exited; no in-flight Smith subagent.

Set `critics_reviewed_ref = git rev-parse HEAD`. Spawn Scout (`subagents/scout.md`) and Hawk (`subagents/hawk.md`) — pass each the verified diff, `shipit-state.yaml` path, and (for Scout) the feature type from `project_context.feature_type`. Do not pass design rationale to Scout — Scout stays black-box.

### 7. Critique loop

Collect findings. Route blockers to Smith. Smith revises (one new commit per revision per `references/audit-invariants.md` Invariant 2). Scout and Hawk re-verify on the new HEAD; update `critics_reviewed_ref`. Per-finding pushback ≤3 re-raises per id. Spiral is derived at check time from finding `history` blocker counts across consecutive revisions (≥2 revisions without blocker decrease). Escalate any of: per-finding 3-pushback exhaustion, Scout↔Hawk conflict on same code, regression spiral.

If Smith's subagent has terminated between revisions, re-dispatch a fresh Smith. Main agent never patches code (see `references/role-boundaries.md`).

### 8. Ship-ready gate

Gate re-derives every condition from **primary artifacts** (git history, filesystem, GitHub PR state), not from `phase_log` or derived counters. See `references/audit-invariants.md` Invariant 4 for rationale and per-phase artifact table.

Main agent MUST mechanically iterate the probe list below and record each probe's verdict + evidence. A `phase_log` entry is not proof; the artifact is proof. Skipping a probe is a gate violation.

**Gate probes (must all pass):**

```
P0a   git -C "$worktree_path" rev-parse --abbrev-ref HEAD ≠ {main,master}; worktree dir exists
P0b   project_context.gh_authenticated == true
P1    every project_context field non-null or explicitly "unavailable" with reason
P3    docs/specs/<slug>/spec.md exists; ≥1 AC item; each AC has branches_required
P5    git log --grep "^Shipit-Revision: 0" "$base_ref"..HEAD returns ≥1 commit
P5.5  verification_report.<final_revision>.yaml exists; verdict == accept
       where final_revision := git log --grep "^Shipit-Revision:" max N
P6/7  Scout + Hawk submission files exist for final_revision
       critics_reviewed_ref == `git -C "$worktree_path" rev-parse HEAD`
       every findings_index entry: status ∈ {fixed, acked}
       every entry with status == escalated: user_resolution != null
P10   `git log --grep "^Shipit-Phase-10: true"` returns ≥1 commit
       OR docs/specs/<slug>/phase-10-waived.txt exists with user signature line
Caps  smith_dispatch_count ≤ 6
       derived verification_failure_streak ≤ 2
       derived spiral_streak ≤ 2
```

If any probe fails:

- P5/P5.5/P6/P7 blocker `open` → route back into Phase 7.
- Blocker `escalated` without `user_resolution` → produce packet per `references/escalation-packet.md`. **"Ship anyway, file follow-up issue" is not a permitted resolution** — `accept_escalated` is the only audit-trailed equivalent.
- Completeness declaration missing in a submission file → finding raised against submitting agent; loop.
- HEAD drift (probe P6/P7 fails on SHA match) → re-run Phase 6 critics on current HEAD.
- P10 missing → run Phase 10 OR produce waiver file (requires explicit user `--skip-cleanup` confirm; main agent does not self-waive).
- Any probe missing primary artifact while `phase_log` claims phase exited → journaling drift; the artifact is authoritative, the journal is wrong. Re-run that phase.

### 9. Ship-ready handoff

**Entry:** Phase 8 returned true.

**9.1 — Create draft PR.**

```bash
cd "$worktree_path"
git push -u "$default_remote" "$feature_branch"
gh pr create --draft \
  --title  "<follows project_context.pr_title_convention>" \
  --body   "<assembled from Scout AC report + Hawk summary + finding log>"
```

Set `run_outcome.pr_url`.

**9.2 — Hand off to user.** Output: PR URL, Scout AC report, Hawk summary, finding log, escalation packet (if any), branch / worktree path, `phase_log` summary.

**9.3 — Forbidden:** `gh pr merge`; `git push --force` (except resolving an explicit escalation with user confirmation); un-drafting the PR.

### 10. Post-ship knowledge cleanup

Prefer `neat-freak`. Updates docs / memory / CLAUDE.md only. **Scope:** non-source-code paths only. If a Phase 10 commit is produced, it carries trailer `Shipit-Phase-10: true` and is exempt from `critics_reviewed_ref` invariance (per `references/audit-invariants.md` Invariant 3). A Phase 10 commit that touches source code triggers a re-run of Phase 6 critics on the new HEAD.

Phase 10 is part of the ship-ready gate. The user may waive it with explicit `--skip-cleanup`.

## Hard rules

These are the only unconditional rules. All other guidance is invariant-with-default-mechanism, documented in the references.

- Never auto-merge. Output is a draft PR, not a merged PR.
- Never work on `main` / `master`.
- Smith must not skip Scout or Hawk. Both run on every feature.
- All critic findings use the structured schema (`references/finding-schema.md`). No prose reviews.
- Subagent submissions are unverified assertions until Phase 5.5 verifies them against repo state.
- The ship-ready gate is a pure function of `shipit-state.yaml`. No re-derivation from chat history.
- User is sole arbiter of escalations in v1. Main agent packages context; main agent does not decide.
- `project_context` is populated once at Phase 1 and is read-only thereafter. Re-derivation at point of use is forbidden.
