# Audit Invariants

shipit maintains three parallel audit trails. Each invariant is reconstructible after the fact: from the finding files, from git history, and from `shipit-state.yaml`. Together they make any past run inspectable — what was found, what was changed in response, when each phase ran.

Rules below are stated as invariants (what must hold) with default mechanisms. Alternatives are acceptable when the invariant still holds; only explicitly forbidden actions are unconditional.

## Invariant 1 — Finding history is preserved

**Invariant:** every finding's full dispute chain (raise → fix / justify / re-raise / accept / escalate) is reconstructible from its `history` array.

**Default mechanism:** every Smith ↔ critic exchange appends one entry to `history` per the schema in `finding-schema.md`. Same finding id reused across re-raises; new issue → new id with `pushback_count: 0` and `differs_from: [prior_ids]` when within 10 lines of a prior finding.

**Forbidden:** id-laundering (cosmetic rename to reset `pushback_count`); silent finding deletion; collapsing multiple exchanges into a single entry.

**Where stored:** `docs/specs/YYYY-MM-DD-<slug>/findings/<id>.yaml`, one file per finding. Index lives in `shipit-state.yaml` → `findings_index`.

## Invariant 2 — Git history is per-revision

**Invariant:** the boundary between Smith revisions is reconstructible from git history. After any squash, the boundary commits remain identifiable by trailer.

**Default mechanism:** revision 0 produces ≥1 new commit; revisions 1..N each produce ≥1 new commit. The **last commit of each revision** carries trailers:

```
Shipit-Revision: <N>
Shipit-Findings-Addressed: <comma-separated finding ids, or "initial" for revision 0>
```

Intermediate commits within a revision (e.g., red / green / refactor under TDD) are fine — they need not carry the trailer.

**Alternative mechanism:** a single commit per revision (the trailer commit is also the only commit). Acceptable.

**Forbidden:** `git commit --amend` across a revision boundary (collapses two revisions into one commit, destroying the audit anchor). Within a single revision, amend is permitted only if the trailer commit has not yet been made.

**Final squash:** at merge time the user may squash all revision commits into one. The squashed commit's body should aggregate `Shipit-Findings-Addressed:` lines. GitHub's default squash-and-merge body preserves the trailers automatically; verify before merge.

## Invariant 3 — Phase log is complete (observability) and the review ref is anchored

**Invariant:** every phase has an exit record in `shipit-state.yaml → phase_log` for observability and resume — **this is not a gate input** (the gate re-derives from primary artifacts per Invariant 4; a missing phase_log entry is a journaling defect, a missing primary artifact is a gate failure). The commit SHA that critics reviewed is anchored; the ship-ready gate verifies HEAD has not drifted since.

**Default mechanism:**

- Each phase appends an entry to `phase_log` on entry; sets `exited_at` + `exit_conditions_met` + `artifacts` on exit. Schema in `state-file.md`.
- Phase 6 sets `critics_reviewed_ref = git -C "$git_anchor.worktree_path" rev-parse HEAD` when Scout + Hawk begin a round.
- Phase 7 updates `critics_reviewed_ref` (same pinned command) each time critics re-verify a Smith revision.
- Phase 8 ship-ready gate requires `git -C "$git_anchor.worktree_path" rev-parse HEAD == critics_reviewed_ref`. If HEAD has moved (any commit since last critic exit), gate fails. Findings are stale; re-dispatch critics on the new HEAD.

**Phase 10 carve-out:** Phase 10 (post-ship knowledge cleanup) may modify docs / memory / CLAUDE.md. If Phase 10 produces a commit, the commit must carry trailer `Shipit-Phase-10: true` and touch ONLY non-source-code paths (docs, memory, top-level README, CLAUDE.md). Such commits are excluded from the `critics_reviewed_ref` invariance check.

**Forbidden:** main agent or any subagent silently committing after critic exit without bumping `critics_reviewed_ref` and re-running critics. A Phase 10 commit that touches source code is an invariant violation; re-run Phase 6 critics on the new HEAD before the gate can pass.

## Invariant 4 — Every actor boundary admits claims only with falsifiable evidence

**Invariant:** every actor boundary in shipit (Smith → Main at Phase 5.5; Main → Gate at Phase 8) admits a claim only when the claiming actor furnishes evidence that could falsify the claim. The downstream actor cross-checks the furnished evidence against an independent probe; it does not re-derive the claim from scratch, and it does not accept the claim on the actor's word. Absence of the falsifiable evidence makes the claim **invalid at the boundary** — not a thing to verify later.

**Rationale:** a self-reporting actor cannot certify its own completion — a budget-exhausted Smith still emits a "done" receipt; a main agent that forgets a phase on the write side forgets it on the read side too. Same actor on both sides of a boundary = self-consistency masquerading as correctness. The fix is not "verify harder downstream" (expensive, post-hoc — a full cycle wasted to reject). The fix is to shift the burden upstream: the claimant attaches the artifact that would prove it wrong if it lied. A budget-killed Smith physically cannot attach passing `$test_command` output (producing it requires running the command), so it cannot emit a false ship-ready claim — only an honest partial. Downstream then does a cheap cross-check, and a *mismatch* between furnished evidence and independent probe is a fabrication finding (more severe than incompleteness).

**Two instantiations of the one rule:**

- **Smith → Main (Phase 5.5):** Smith furnishes `verification_evidence` (raw `$test_command` tail). Missing / `tests_passed != true` → invalid at boundary. Furnished but contradicted by re-run → fabrication blocker.
- **Main → Gate (Phase 8 → 9):** the main agent furnishes `gate_evidence.yaml` — per probe, the exact command and its **verbatim stdout** plus `pass`. Phase 9 entry rejects the gate if the block is absent, any `pass != true`, or any verdict lacks captured stdout (`gate_unverified`). The actor asserting the gate passed attaches the artifact that would falsify it; a bare `phase_log: "gate passed"` is the self-report this forbids.

Both instantiations must carry a mandatory falsifying artifact. A statement on one side and an artifact on the other is asymmetric — the stated side degrades to self-report.

**Primary artifact per required phase:**

Every git probe below is pinned `git -C git_anchor.worktree_path`. An unpinned probe reads the ambient repo (possibly on `master`).

| Phase | Primary artifact (gate probes this, not `phase_log`) |
|---|---|
| 0a   | worktree exists at `git_anchor.worktree_path`; current branch ≠ `main`/`master` |
| PG   | `rev-parse --abbrev-ref HEAD == git_anchor.feature_branch` (recorded branch is real); `rev-parse "$base_ref" == git_anchor.base_sha` (base has not advanced — no feature commit on `master`); no two `"$base_ref"..HEAD` commits share a `Shipit-Revision` value (no cherry-pick/recommit duplicates) |
| 0b   | `gh auth status` ok; `project_context.gh_authenticated == true` |
| 1    | every `project_context` field non-null (or explicitly `unavailable` with reason) |
| 2    | `phase_log` entry present (intent clarification has no on-disk artifact — accept journal entry here) |
| 3    | `docs/specs/<slug>/spec.md` exists; ≥1 AC item; each AC has `branches_required` |
| 4    | plan artifact present in spec dir, OR `phase_log` entry recording "no split needed" with token estimate |
| 5    | ≥1 commit on `git_anchor.feature_branch` with `Shipit-Revision: 0` trailer |
| 5.5  | `verification_report.<N>.yaml` exists for highest revision N; verdict `accept` |
| 6/7  | per-revision Scout + Hawk submission files exist; `critics_reviewed_ref == git -C $wt rev-parse HEAD`; all `findings_index` entries have `status ∈ {fixed, acked}` with `user_resolution` set on any `escalated` |
| 9    | `gh pr view "$pr_url"` returns; PR state = draft |
| 10   | commit with trailer `Shipit-Phase-10: true` exists OR waiver file `docs/specs/<slug>/phase-10-waived.txt` exists with user signature |

**Derived counters:** values computable from primary artifacts MUST NOT be stored as authoritative fields in the state file. The state file may cache them for performance, but the gate re-derives at evaluation time. Specifically:

- `final_revision` ← `git log --grep "^Shipit-Revision:" "$base_ref"..HEAD` max N
- `verification_failure_streak` ← count trailing non-`accept` verdicts (`reject` / `fabrication`) in `verification_report.*.yaml`
- `spiral_streak` ← count trailing revisions with non-decreasing blocker count, computed from finding `history` arrays

`smith_dispatch_count` has no repo-side artifact (subagent spawns leave no trace) — kept as authoritative counter, but bounded by independent check at re-dispatch sites.

**Forbidden:** gate accepting `phase_log` entry as proof of phase completion without probing the corresponding primary artifact. Gate trusting any derived counter read from state file without re-derivation.

## Why four invariants, not one

Each invariant covers a distinct failure mode:

- **Without Invariant 1**, critics drift to prose reviews; pushback budget is unenforceable; escalations have no context.
- **Without Invariant 2**, "ship this with a quick fixup amend" silently collapses the revision sequence — fine until you need to bisect a regression to a specific revision.
- **Without Invariant 3**, "main agent fixed a small thing in Phase 10" silently ships unreviewed code under a green gate.
- **Without Invariant 4**, a self-reporting actor certifies its own completion: a budget-exhausted Smith emits a false "done" and wastes a full verify cycle; the gate inherits the main agent's blind spots and returns `ship_ready: true` under the same omission that created the gap.

All four failure modes have happened in practice. The invariants exist because the failures happened, not because they sound rigorous.
