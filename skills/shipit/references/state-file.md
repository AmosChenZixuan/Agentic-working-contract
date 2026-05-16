# Run State File (`shipit-state.yaml`)

shipit's single source of truth for a run. All phases read from and write to this file. Conversation context is volatile (compaction, agent restart, subagent dispatch loses parent context); the state file survives.

## Location

```
docs/specs/YYYY-MM-DD-<slug>/shipit-state.yaml
```

Sibling of the spec document. One file per shipit run. Created at Phase 0a, finalized at Phase 9.

If `docs/specs/YYYY-MM-DD-<slug>/` does not exist, create it at Phase 0a.

## Schema

```yaml
slug:        <feature-slug>
spec_path:   docs/specs/YYYY-MM-DD-<slug>/spec.md
created_at:  <iso8601>

# ─── Git anchor (probed ground truth) ───────────────────────────────
# Written ONCE at Phase 0a by probing the worktree, never by intent.
# The branch-creating tool may name the branch differently than asked
# (e.g. worktree-feat+X vs feat/X) — this records what git actually says.
# Every later git probe (Phase 5.5, 8) is pinned to these values and
# fails closed on mismatch. No phase may overwrite from inference.
git_anchor:
  workspace_mode:    worktree | branch                   # the decision, recorded
  workspace_chosen_by: user | prompt                      # user-instructed vs Phase 0a prompt
  base_ref:        <ref the workspace branched from, e.g. origin/main — set at 0a, not Phase 1>
  worktree_path:   <absolute path>                       # never relative; repo root in branch mode
  feature_branch:  <git -C "$wt" rev-parse --abbrev-ref HEAD AT Phase 0a>
  base_sha:        <git -C "$wt" rev-parse "$base_ref"   AT Phase 0a>

# ─── Project context ────────────────────────────────────────────────
# Populated once at Phase 1 by main agent. Read-only afterward.
# All later phases / subagents read from here; none re-derive.
project_context:
  worktree_path:        <= git_anchor.worktree_path; copied, never re-derived>
  base_ref:             <= git_anchor.base_ref; copied at Phase 1, never re-detected>
  feature_branch:       <= git_anchor.feature_branch; copied, never re-derived>
  default_remote:       <e.g., origin>
  gh_authenticated:     true | false                     # `gh auth status`
  test_command:         <e.g., "pytest -v", "npm test", "cargo test">
  lint_command:         <e.g., "ruff check .", "eslint .">  # null if none discovered
  commit_convention:    <one-line description, e.g., "Conventional Commits", "[scope] subject", "中文 imperative">
  commit_convention_evidence: <git log line that demonstrates, or path to CONTRIBUTING>
  pr_title_convention:  <one-line description; null if no merged PRs to infer from>
  feature_type:         web-ui | http-api | cli | library | mixed
  feature_type_evidence: <one-line reason>

# ─── Phase log (narrative journal) ──────────────────────────────────
# Append-only journal for observability and resume. NOT the gate input —
# the ship-ready gate (Phase 8) re-derives phase completion from primary
# artifacts per audit-invariants Invariant 4. A missing phase_log entry
# is a journaling defect; a missing primary artifact is a gate failure.
phase_log:
  - phase: <0a | 0b | 1 | 2 | 3 | 4 | 5 | 5.5 | 6 | 7 | 8 | 9 | 10 | 11>
    entered_at: <iso8601>
    entry_conditions_met: [<conditions actually checked>]
    exited_at:  <iso8601 or null>                        # null while in progress
    exit_conditions_met:  [<conditions actually checked>]
    artifacts:  [<paths or refs produced this phase>]
    notes:      <optional>

# ─── Anchored review ref ────────────────────────────────────────────
# Set by Phase 6 when critics begin. Updated by Phase 7 every time
# critics re-review after a Smith revision. The ship-ready gate (Phase 8)
# requires `HEAD == critics_reviewed_ref`. If HEAD moved after critic exit,
# findings are stale — gate fails.
critics_reviewed_ref:  <git commit sha>
critics_reviewed_at:   <iso8601>

# ─── Counters ───────────────────────────────────────────────────────
# Only authoritative counter is smith_dispatch_count (no repo-side artifact
# for subagent spawns). All other counters are DERIVED from primary
# artifacts and MUST be re-derived at gate time, not read from this file.
# See references/audit-invariants.md Invariant 4.
counters:
  smith_dispatch_count:           <integer>              # incremented every Smith spawn; cap = 6
  agent_ids:                                              # persisted-subagent handles; SendMessage targets.
    smith:  <handle | null>                               # process-local: null/stale after a main-session
    scout:  <handle | null>                               # resume → that role takes the cold path.
    hawk:   <handle | null>                               # See role-boundaries § Subagent continuity.
  # Derived (do not store as authoritative; gate re-derives):
  #   final_revision               ← git log --grep "^Shipit-Revision:" max N
  #   verification_failure_streak  ← count trailing non-`accept` in verification_report.*.yaml; cap = 2
  #   spiral_streak                ← count trailing revisions w/ non-decreasing blockers; cap = 2

# ─── Findings index ─────────────────────────────────────────────────
# One entry per finding raised in this run. Full finding objects live in
# findings/<id>.yaml under the spec directory; this index is the
# ship-ready gate's input.
findings_index:
  - id:              <finding_id>
    raised_by:       Scout | Hawk
    severity:        blocker | advisory
    status:          open | fixed | acked | escalated
    user_resolution: <null | accept_escalated | override_critic | side_with_critic | revise_ac | revise_spec | abort>
    path:            findings/<id>.yaml

# ─── Run outcome ────────────────────────────────────────────────────
run_outcome:
  status:  in_progress | ship_ready | aborted
  pr_url:  <null until Phase 9.1>
  reason:  <set on abort>
```

## Read / write discipline

- **Phase 0a** creates the file with `slug`, `spec_path`, `created_at`, the first `phase_log` entry (its `notes` stating the workspace decision + why), and `git_anchor`: `workspace_mode`/`workspace_chosen_by`/`base_ref` from the recorded decision (`base_ref` is set here, not Phase 1 — workspace creation needs it), `feature_branch`/`base_sha`/`worktree_path` by probing the workspace (`rev-parse --abbrev-ref HEAD`, `rev-parse "$base_ref"`). `git_anchor` is write-once; no later phase may overwrite it from inference. A later probe disagreeing with `git_anchor` is a git-surface anomaly → escalate per `references/role-boundaries.md`, do not "fix up" history.
- **Phase 1** populates `project_context` in full. After Phase 1 exits, `project_context` is read-only — later phases that need a value MUST read it from here. They MUST NOT re-derive (no second `git log` for commit convention, no second `pytest --collect-only`, no second `gh auth status`). If a value is missing from `project_context`, that is a Phase 1 defect — go back to Phase 1, do not work around.
- **Each phase entry** appends to `phase_log` on entry, sets `exited_at` + `exit_conditions_met` + `artifacts` on exit.
- **Phase 5.5** admits Smith's submission only if `verification_evidence` is present with `tests_passed: true`, then cross-checks it against one independent re-run. Persists Smith's raw submission to `submission.<N>.yaml` (Smith subagent output is otherwise non-recoverable; cold re-dispatch + Phase 11 read it) and writes `verification_report.<N>.yaml` (verdict `accept` | `reject` | `fabrication` + evidence + re-run output) per Smith revision. Failure streak is derived from these files (trailing non-`accept`), not stored.
- **Phase 6** sets `critics_reviewed_ref = git -C "$git_anchor.worktree_path" rev-parse HEAD` when critics begin. Spawns Scout + Hawk once, records `counters.agent_ids.{scout,hawk}`; Smith stays alive (not terminated to start critics). Writes Scout + Hawk submission files keyed by revision N. Persistence/cold-respawn per role-boundaries § Subagent continuity.
- **Phase 7** updates `critics_reviewed_ref` on every Smith re-revision + critic re-verify cycle. Updates `findings_index`. Bumps `counters.smith_dispatch_count` on each Smith re-dispatch. Spiral streak is derived from finding `history` blocker counts, not stored.
- **Phase 8 ship-ready gate** re-derives every condition from primary artifacts per audit-invariants Invariant 4, every git probe pinned `git -C git_anchor.worktree_path`. It does NOT trust `phase_log` entries or derived counters as proof. Writes `docs/specs/<slug>/gate_evidence.yaml`: one entry per probe = `{probe, command, stdout (verbatim), pass}`. Per-phase artifact probes + the `PG` git-substrate probe are listed in audit-invariants § Invariant 4.
- **Phase 9** admits the gate only if `gate_evidence.yaml` exists with every `pass: true` and every probe carrying captured stdout (missing block / unproven pass → `gate_unverified`, loop back). Then sets `run_outcome.pr_url` after `gh pr create --draft`.
- **Phase 10** edits to docs / memory / CLAUDE.md DO NOT move HEAD inside the worktree's source tree. If Phase 10 produces a commit, that commit must be marked `Shipit-Phase-10: true` in trailers and excluded from the `critics_reviewed_ref` invariance check, OR Phase 10 changes must be reviewed by re-running Phase 6 critics on the new HEAD.
- **Phase 11** writes `docs/specs/<slug>/session-reflection.md` by harvesting each participant's `reflection` block from the last persisted artifact (`submission.<final_revision>.yaml`; each critic's final-round file) plus the Main orchestrator's own section. Not a gate probe. A participant with no persisted reflection is recorded `unavailable — <reason>`, never fabricated.

## Why a file, not conversation state

Conversation context is unreliable substrate for state:

- **Compaction destroys it.** Long runs hit the context window; summarization paraphrases away exact field values.
- **Subagent dispatch fragments it.** Smith / Scout / Hawk each get fresh context windows. Without a file, every dispatch re-explains the run.
- **Agent restart wipes it.** A new session can pick up an in-progress run by reading `shipit-state.yaml` — without it, the run cannot resume.
- **Gates need a substrate they can read deterministically.** "Did Phase 5.5 exit?" is a one-line file lookup. "Did the conversation mention Phase 5.5 exiting?" is fuzzy retrieval prone to hallucination.

The state file is to shipit what a Makefile's `.PHONY` targets and timestamps are to make: not the work itself, but the bookkeeping that makes the work auditable and resumable.
