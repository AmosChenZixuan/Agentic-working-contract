---
name: shipit
description: Use when delivering a single PR-scoped feature end-to-end with multi-critic review loops. Triggers on "/shipit", "ship this feature", or any request for full-cycle feature delivery from design through ship-ready PR with structured critique.
metadata:
  version: 0.0.9
---

# shipit

Orchestrator skill. Main agent owns design, AC, planning, gate evaluation, and arbitration. Spawns three named subagents per feature:

- **Smith** — white-box implementer; forges the feature end-to-end (design → tests → code → refactor); keeps full feature context with no mid-flight compaction.
- **Scout** — black-box AC verifier; reconnoiters from outside; exercises the feature via the appropriate tool (playwright MCP for web UI, curl / shell / test-runner otherwise) without reading design rationale.
- **Hawk** — white-box code reviewer; sharp eyes for correctness, safety, regression, perf, style.

Scout and Hawk run in parallel after Smith submits and feed structured findings back to Smith in a bounded critique loop. Output is a ship-ready draft PR — never auto-merged.

## Core principles

- **Context-boundary over task-boundary.** Every subagent (Smith, Scout, Hawk) is spawned once and persists for the whole run, contacted via `SendMessage`; cold re-spawn only if its handle is unrecoverable, under a reconstruction obligation. See `references/role-boundaries.md` § Subagent continuity. Eliminates inter-agent re-derivation cost.
- **Single PR scope.** Smith's initial loaded context ≤60k tokens. If diff likely >800 LOC or touches >8 files, split feature upstream during planning.
- **Parallel critics, structured findings only.** Scout + Hawk run in parallel after Smith's first submission. All findings use the schema in `references/finding-schema.md`. No prose reviews.
- **Per-finding pushback budget = 3.** Critic re-raises same finding id at most 3 times. Then that finding escalates. Loop continues on other findings until no blockers remain.
- **Spiral detector.** Two consecutive Smith revisions without blocker-count strictly decreasing → escalate.
- **Never auto-ship.** End state is a draft PR. User merges manually.
- **Never on main.** Smith refuses to work on `main` / `master`. Workspace mode (worktree vs branch) is a recorded Phase 0a decision — user instruction if given, else prompt; never a silent tooling default.
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

shipit is a state machine. Each phase has entry conditions (must hold to start) and exit conditions (must hold to leave). Every phase appends an entry to `shipit-state.yaml → phase_log` for observability and resume — **the ship-ready gate does not consume the log; it re-derives every condition from primary artifacts** (`references/audit-invariants.md` Invariant 4). Phases cannot be skipped, reordered, or marked complete without their exit conditions actually holding.

### 0a. Branch / worktree check

- **Determine `base_ref` first** — the ref the workspace branches from (default the repo's main ref, e.g. `origin/main`; honor an explicit user override). Workspace creation and the anchor both need it, so it is established here, not at Phase 1. Phase 1 only copies it.
- **Workspace mode is a recorded decision with no default.** shipit has no worktree-vs-branch preference. If the user already stated one, honor it. Otherwise you MUST ask before creating anything — prompt `[branch / worktree / abort]`. Never pick one because a tool happens to be installed.
  - `branch` → `git checkout -b <branch>` from `base_ref`.
  - `worktree` → `superpowers:using-git-worktrees` if available, else `git worktree add`.
  - On `main` / `master` with no stated mode: the same prompt, `abort` available.
- Create `docs/specs/YYYY-MM-DD-<slug>/` and initialize `shipit-state.yaml` with the first phase_log entry.
- **Record the git anchor by probing the workspace — never by intent.** The workspace tool may name the branch differently than requested (`worktree-feat+X` vs `feat/X`). Probe and record ground truth:

```bash
wt=<absolute worktree path>
git -C "$wt" rev-parse --abbrev-ref HEAD          # → git_anchor.feature_branch
git -C "$wt" rev-parse "$base_ref"                # → git_anchor.base_sha
```

  Write `git_anchor.{workspace_mode, workspace_chosen_by, base_ref, worktree_path, feature_branch, base_sha}` to `shipit-state.yaml`. `worktree_path` is the working directory (worktree dir, or repo root in `branch` mode). This block is the falsifiable anchor for every later git probe. Phase 1 and all later phases read `base_ref` / `feature_branch` **from here**, not from the slug or the requested name; none may overwrite from inference. `phase_log[0].notes` states the mode chosen and why (user-instructed vs prompted).

**Exit:** `shipit-state.yaml` exists; `phase_log[0]` records the workspace decision; `git_anchor` written from probes (not intent); `pwd` is the workspace.

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

**Populate `shipit-state.yaml → project_context` in full.** All later phases and subagents read from here; none re-derive. `base_ref`, `worktree_path`, `feature_branch`, and `base_sha` are NOT detected here — they live in `git_anchor` (set at Phase 0a) and are copied, never re-inferred. Detect and record:

- `base_ref` — copied from `git_anchor.base_ref` (must resolve to `git_anchor.base_sha`)
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

Spawn Smith once with `subagents/smith.md`. Pass: spec, AC, plan, design decisions log, and the path to `shipit-state.yaml` (Smith reads `git_anchor.worktree_path`, `test_command`, etc. from there). Bump `counters.smith_dispatch_count`; record the handle in `counters.agent_ids.smith`. All later Smith contact is `SendMessage` per `references/role-boundaries.md` § Subagent continuity.

**Exit:** Smith returns a submission schema. Submission is an **assertion** at this point — Phase 5.5 verifies it before it counts as fact.

### 5.5. Submission verification (cross-check, non-skippable)

Smith's submission must carry its own falsifiable evidence (`verification_evidence` block — see `subagents/smith.md`). Phase 5.5 does NOT re-derive completion from scratch; it cross-checks Smith's attached evidence against one independent re-run. This is `references/audit-invariants.md` Invariant 4 applied to the Smith → Main boundary.

**Boundary admission check (before any work):**

- `verification_evidence` block absent, OR `tests_passed != true`, OR `status: incomplete` → submission is **invalid at the boundary** (`rejected_unverified`). Not a thing to verify — a thing that was never a valid ship-ready submission. Re-dispatch Smith (incremental — see `references/role-boundaries.md`).

**Cross-check (only if admitted):** `wt := git_anchor.worktree_path`:

```bash
git -C "$wt" rev-parse --abbrev-ref HEAD              # must == git_anchor.feature_branch
git -C "$wt" rev-parse "$base_ref"                    # must == git_anchor.base_sha (base unmoved)
git -C "$wt" log -1 --format='%H %(trailers:key=Shipit-Revision)'   # head_sha + trailer
git -C "$wt" status --porcelain
git -C "$wt" diff --name-only "$base_ref"...HEAD      # files actually changed
git -C "$wt" log --oneline "$base_ref"..HEAD          # commits actually made
$test_command                                         # ONE independent re-run
```

Compare against Smith's `verification_evidence`:

- Re-run matches attached evidence AND diff matches claimed `files_touched` AND branch == `git_anchor.feature_branch` AND base == `git_anchor.base_sha` AND `Shipit-Revision` trailer present on HEAD → `accept`.
- Re-run **contradicts** attached evidence (Smith reported pass, re-run fails; or summary counts diverge) → **integrity blocker**: Smith fabricated evidence. More severe than incompleteness — record `fabrication: true`, escalate immediately (do not re-dispatch silently).
- Branch ≠ `git_anchor.feature_branch`, base advanced past `git_anchor.base_sha`, or HEAD missing its `Shipit-Revision` trailer → **git-surface anomaly**: do NOT cherry-pick/amend to repair. Escalate per `references/role-boundaries.md` § git-surface anomaly.
- Diff/commit mismatch (claimed files not in diff) → `rejected_unverified`, re-dispatch incremental.

Main agent's only allowed actions are **accept**, **re-dispatch Smith incrementally**, or **escalate**. Main agent does NOT patch the gap itself (`references/role-boundaries.md` — no triviality exception).

Persist Smith's raw submission verbatim to `docs/specs/<slug>/submission.<N>.yaml` (the on-disk artifact that cold re-dispatch and Phase 11 read — Smith's subagent output is otherwise non-recoverable). Then write `verification_report.<N>.yaml` (verdict `accept` | `reject` | `fabrication` + evidence + the re-run output) per dispatch. `verification_failure_streak` is derived at check time as the count of trailing non-`accept` verdicts. If streak ≥ 2 or `smith_dispatch_count` ≥ 6, escalate to user.

**Exit:** evidence admitted and cross-check `accept`; re-check Phase 4's size heuristic against actual diff (escalate if exceeded); `verification_report` artifact written.

### 6. Dispatch Scout + Hawk in parallel

**Entry:** Phase 5.5 exited; Smith idle (submission accepted, not mid-revision) — Smith's subagent stays alive per § Subagent continuity, not terminated to start critics.

Set `critics_reviewed_ref = git -C "$git_anchor.worktree_path" rev-parse HEAD`. Spawn Scout (`subagents/scout.md`) and Hawk (`subagents/hawk.md`) **once each**; record handles in `counters.agent_ids.{scout,hawk}`. Pass each the verified diff, `shipit-state.yaml` path, and (for Scout) `project_context.feature_type`. Do not pass design rationale to Scout — Scout stays black-box. Re-verification in Phase 7 is `SendMessage` to these handles, not re-spawns (§ Subagent continuity).

### 7. Critique loop

Collect findings. `SendMessage` blockers to the persisted Smith; it revises (one new commit per revision per `references/audit-invariants.md` Invariant 2). `SendMessage` the persisted Scout and Hawk `re-verify HEAD <sha>`; update `critics_reviewed_ref`. All three contacts follow `references/role-boundaries.md` § Subagent continuity (persisted handle; cold re-spawn only if unrecoverable; main agent never patches code). Per-finding pushback ≤3 re-raises per id. Spiral is derived at check time from finding `history` blocker counts across consecutive revisions (≥2 revisions without blocker decrease). Escalate any of: per-finding 3-pushback exhaustion, Scout↔Hawk conflict on same code, regression spiral.

### 8. Ship-ready gate

Gate re-derives every condition from **primary artifacts** (git history, filesystem, GitHub PR state), not from `phase_log` or derived counters. See `references/audit-invariants.md` Invariant 4 for rationale and per-phase artifact table.

Every git probe is pinned `git -C "$wt"`, `wt := git_anchor.worktree_path`. An unpinned `git` reads the ambient repo (possibly on `master`) and is itself a gate violation.

Main agent runs each probe below and captures its **verbatim stdout** into `gate_evidence` (Phase 9 entry). The captured output is proof; a `phase_log` entry is not. A verdict without its stdout is a skipped probe.

**Gate probes (must all pass; `wt := git_anchor.worktree_path`):**

```
P0a   git -C "$wt" rev-parse --abbrev-ref HEAD ≠ {main,master}; worktree dir exists
PG    git substrate matches the probed anchor (catches wrong-context commits):
       git -C "$wt" rev-parse --abbrev-ref HEAD == git_anchor.feature_branch
       git -C "$wt" rev-parse "$base_ref" == git_anchor.base_sha   (base has NOT advanced;
         a feature commit landed on base/master ⇒ this fails)
       git -C "$wt" log --format=%H "$base_ref"..HEAD has no commit with the same
         Shipit-Revision value as another (no cherry-pick/recommit duplicates)
P0b   project_context.gh_authenticated == true
P1    every project_context field non-null or explicitly "unavailable" with reason
P3    docs/specs/<slug>/spec.md exists; ≥1 AC item; each AC has branches_required
P5    git -C "$wt" log --grep "^Shipit-Revision: 0" "$base_ref"..HEAD returns ≥1 commit
P5.5  verification_report.<final_revision>.yaml exists; verdict == accept
       where final_revision := git -C "$wt" log --grep "^Shipit-Revision:" max N
P6/7  Scout + Hawk submission files exist for final_revision
       critics_reviewed_ref == git -C "$wt" rev-parse HEAD
       every findings_index entry: status ∈ {fixed, acked}
       every entry with status == escalated: user_resolution != null
P10   git -C "$wt" log --grep "^Shipit-Phase-10: true" returns ≥1 commit
       OR docs/specs/<slug>/phase-10-waived.txt exists with user signature line
Caps  smith_dispatch_count ≤ 6
       derived verification_failure_streak ≤ 2
       derived spiral_streak ≤ 2
```

A `PG` failure is a **git-surface anomaly**: stop, escalate per `references/role-boundaries.md` § git-surface anomaly (no history surgery).

If any probe fails:

- P5 missing (`Shipit-Revision: 0` commit absent) → re-dispatch Smith (not a finding; nothing to review yet).
- P5.5/P6/P7 blocker `open` → route back into Phase 7.
- Blocker `escalated` without `user_resolution` → produce packet per `references/escalation-packet.md`. **"Ship anyway, file follow-up issue" is not a permitted resolution** — `accept_escalated` is the only audit-trailed equivalent.
- Completeness declaration missing in a submission file → finding raised against submitting agent; loop.
- HEAD drift (probe P6/P7 fails on SHA match) → re-run Phase 6 critics on current HEAD.
- P10 missing → run Phase 10 OR produce waiver file (requires explicit user `--skip-cleanup` confirm; main agent does not self-waive).
- Any probe missing primary artifact while `phase_log` claims phase exited → journaling drift; the artifact is authoritative, the journal is wrong. Re-run that phase.

### 9. Ship-ready handoff

**Entry (boundary admission, symmetric to Phase 5.5):** `docs/specs/<slug>/gate_evidence.yaml` exists with, per Phase 8 probe, the exact command, its verbatim stdout, and `pass: true|false`. Absent block, any `pass != true`, or any probe lacking captured stdout → `gate_unverified`: loop back, do not create the PR. This is Invariant 4 (`references/audit-invariants.md`) at the Main → Gate boundary.

**9.1 — Create draft PR.**

```bash
wt="$git_anchor.worktree_path"
git -C "$wt" push -u "$default_remote" "$git_anchor.feature_branch"
( cd "$wt" && gh pr create --draft \
    --title  "<follows project_context.pr_title_convention>" \
    --body   "<assembled from Scout AC report + Hawk summary + finding log>" )
# gh has no -C; the subshell cd is scoped and does not leak to later git calls
```

Set `run_outcome.pr_url`.

**9.2 — Hand off to user.** Output: PR URL, Scout AC report, Hawk summary, finding log, escalation packet (if any), branch / worktree path, `phase_log` summary.

**9.3 — Forbidden:** `gh pr merge`; `git push --force` (except resolving an explicit escalation with user confirmation); un-drafting the PR.

### 10. Post-ship knowledge cleanup

Prefer `neat-freak`. Updates docs / memory / CLAUDE.md only. **Scope:** non-source-code paths only. If a Phase 10 commit is produced, it carries trailer `Shipit-Phase-10: true` and is exempt from `critics_reviewed_ref` invariance (per `references/audit-invariants.md` Invariant 3). A Phase 10 commit that touches source code triggers a re-run of Phase 6 critics on the new HEAD.

Phase 10 is part of the ship-ready gate. The user may waive it with explicit `--skip-cleanup`.

### 11. Session reflection (always runs; not a gate probe)

Runs after Phase 10 (or after Phase 9 if Phase 10 was waived). Produces `docs/specs/<slug>/session-reflection.md` — a by-role retro of the run, then surfaced to the user.

A critic cannot know which of its rounds is final (convergence is decided later). So every Scout/Hawk round carries a `reflection` block and Smith carries one in every submission; **Phase 11 uses the last one persisted** — `submission.<final_revision>.yaml` and each critic's final-round file (already on disk from Phases 5.5/6/7). Main agent harvests those and writes its own `Main (orchestrator)` section. Each section is:

- **did** — what this role actually did this run (terse bullets).
- **lesson** — the honest lesson(s), **including misses and what it would do differently**. A reflection that lists only successes is incomplete — re-request it.

Collection discipline (same falsifiability rule as everywhere else):

- Do **not** re-spawn a terminated subagent to obtain a reflection. If a participant terminated (cold path) before emitting one, record `reflection: unavailable — <reason>`. Never fabricate a dead agent's reflection.
- A still-live participant (`counters.agent_ids.<role>`) missing a reflection is reached by `SendMessage`, not a fresh spawn.

**Exit:** `session-reflection.md` exists with one section per participating role (or an explicit `unavailable` reason); reflection surfaced to user.

## Hard rules

These are the only unconditional rules. All other guidance is invariant-with-default-mechanism, documented in the references.

- Never auto-merge. Output is a draft PR, not a merged PR.
- Never work on `main` / `master`.
- Scout and Hawk run on every feature. No directive (including a reflection / post-mortem request) waives them; skipping is a hard-rule violation, not a judgment call.
- All critic findings use the structured schema (`references/finding-schema.md`). No prose reviews.
- A subagent submission without falsifiable evidence attached (`verification_evidence`, `tests_passed: true`) is invalid at the Phase 5.5 boundary — not an assertion to verify later. See `references/audit-invariants.md` Invariant 4.
- The ship-ready gate re-derives every condition from primary artifacts (git / filesystem / PR), every git probe pinned to `git_anchor.worktree_path`, and emits `gate_evidence` (verbatim probe stdout). Phase 9 rejects an evidence-less gate as Phase 5.5 rejects an evidence-less Smith submission. It does NOT trust `shipit-state.yaml` fields or chat history as proof.
- `git_anchor` (branch + base SHA + worktree) is probed once at Phase 0a from real git, never from intent. A later disagreement is a git-surface anomaly: escalate; recovering it by cherry-pick / amend / rebase / reset is forbidden.
- No triviality exception: main agent never edits code/tests in phases 5–7, regardless of change size. Re-dispatch Smith incrementally instead.
- User is sole arbiter of escalations in v1. Main agent packages context; main agent does not decide.
- `project_context` is populated once at Phase 1 and is read-only thereafter. Re-derivation at point of use is forbidden.
