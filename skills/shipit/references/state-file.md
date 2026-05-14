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

# ─── Project context ────────────────────────────────────────────────
# Populated once at Phase 1 by main agent. Read-only afterward.
# All later phases / subagents read from here; none re-derive.
project_context:
  worktree_path:        <absolute path>                  # never relative
  base_ref:             <git ref the worktree branched from, e.g., origin/main>
  feature_branch:       <branch name>
  default_remote:       <e.g., origin>
  gh_authenticated:     true | false                     # `gh auth status`
  test_command:         <e.g., "pytest -v", "npm test", "cargo test">
  lint_command:         <e.g., "ruff check .", "eslint .">  # null if none discovered
  commit_convention:    <one-line description, e.g., "Conventional Commits", "[scope] subject", "中文 imperative">
  commit_convention_evidence: <git log line that demonstrates, or path to CONTRIBUTING>
  pr_title_convention:  <one-line description; null if no merged PRs to infer from>
  feature_type:         web-ui | http-api | cli | library | mixed
  feature_type_evidence: <one-line reason>

# ─── Phase log (state machine) ──────────────────────────────────────
# Append-only. Every required phase must have an entry with exited_at set
# before the ship-ready gate can pass.
phase_log:
  - phase: <0a | 0b | 1 | 2 | 3 | 4 | 5 | 5.5 | 6 | 7 | 8 | 9 | 10>
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
# Global caps prevent unbounded re-dispatch loops.
counters:
  smith_dispatch_count:           <integer>              # incremented every Smith spawn; cap = 6
  smith_revision_count:           <integer>              # increments on each accepted revision
  verification_failure_streak:    <integer>              # consecutive Phase 5.5 rejects; cap = 2
  spiral_streak:                  <integer>              # consecutive revisions w/o blocker decrease; cap = 2

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

- **Phase 0a** creates the file with `slug`, `spec_path`, `created_at`, and the first `phase_log` entry.
- **Phase 1** populates `project_context` in full. After Phase 1 exits, `project_context` is read-only — later phases that need a value MUST read it from here. They MUST NOT re-derive (no second `git log` for commit convention, no second `pytest --collect-only`, no second `gh auth status`). If a value is missing from `project_context`, that is a Phase 1 defect — go back to Phase 1, do not work around.
- **Each phase entry** appends to `phase_log` on entry, sets `exited_at` + `exit_conditions_met` + `artifacts` on exit.
- **Phase 5.5** runs verification against `project_context.base_ref` and `project_context.test_command`. Bumps `counters.verification_failure_streak` on failure.
- **Phase 6** sets `critics_reviewed_ref = git rev-parse HEAD` when critics begin.
- **Phase 7** updates `critics_reviewed_ref` on every Smith re-revision + critic re-verify cycle. Updates `findings_index`. Bumps `counters.smith_dispatch_count` on each Smith re-dispatch; bumps `counters.spiral_streak` when blocker count fails to decrease.
- **Phase 8 ship-ready gate** is a pure function of this file. No reading chat history. No re-derivation. The gate consumes:
  - every entry of `phase_log` has `exited_at` set
  - `critics_reviewed_ref == git rev-parse HEAD` (no drift since last critic exit)
  - every `findings_index` entry's `status ∈ {fixed, acked}` (with `user_resolution` set on any `escalated → acked`)
  - no counter exceeds its cap
- **Phase 9** sets `run_outcome.pr_url` after `gh pr create --draft`.
- **Phase 10** edits to docs / memory / CLAUDE.md DO NOT move HEAD inside the worktree's source tree. If Phase 10 produces a commit, that commit must be marked `Shipit-Phase-10: true` in trailers and excluded from the `critics_reviewed_ref` invariance check, OR Phase 10 changes must be reviewed by re-running Phase 6 critics on the new HEAD.

## Why a file, not conversation state

Conversation context is unreliable substrate for state:

- **Compaction destroys it.** Long runs hit the context window; summarization paraphrases away exact field values.
- **Subagent dispatch fragments it.** Smith / Scout / Hawk each get fresh context windows. Without a file, every dispatch re-explains the run.
- **Agent restart wipes it.** A new session can pick up an in-progress run by reading `shipit-state.yaml` — without it, the run cannot resume.
- **Gates need a substrate they can read deterministically.** "Did Phase 5.5 exit?" is a one-line file lookup. "Did the conversation mention Phase 5.5 exiting?" is fuzzy retrieval prone to hallucination.

The state file is to shipit what a Makefile's `.PHONY` targets and timestamps are to make: not the work itself, but the bookkeeping that makes the work auditable and resumable.
