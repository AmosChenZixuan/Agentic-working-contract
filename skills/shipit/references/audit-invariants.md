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

## Invariant 3 — Phase log is complete and the review ref is anchored

**Invariant:** every phase required by the ship-ready gate has an exit record in `shipit-state.yaml → phase_log`. The commit SHA that critics reviewed is anchored; the ship-ready gate verifies HEAD has not drifted since.

**Default mechanism:**

- Each phase appends an entry to `phase_log` on entry; sets `exited_at` + `exit_conditions_met` + `artifacts` on exit. Schema in `state-file.md`.
- Phase 6 sets `critics_reviewed_ref = git rev-parse HEAD` when Scout + Hawk begin a round.
- Phase 7 updates `critics_reviewed_ref` each time critics re-verify a Smith revision.
- Phase 8 ship-ready gate requires `git rev-parse HEAD == critics_reviewed_ref`. If HEAD has moved (any commit since last critic exit), gate fails. Findings are stale; re-dispatch critics on the new HEAD.

**Phase 10 carve-out:** Phase 10 (post-ship knowledge cleanup) may modify docs / memory / CLAUDE.md. If Phase 10 produces a commit, the commit must carry trailer `Shipit-Phase-10: true` and touch ONLY non-source-code paths (docs, memory, top-level README, CLAUDE.md). Such commits are excluded from the `critics_reviewed_ref` invariance check.

**Forbidden:** main agent or any subagent silently committing after critic exit without bumping `critics_reviewed_ref` and re-running critics. A Phase 10 commit that touches source code is an invariant violation; re-run Phase 6 critics on the new HEAD before the gate can pass.

## Why three invariants, not one

Each invariant covers a distinct failure mode:

- **Without Invariant 1**, critics drift to prose reviews; pushback budget is unenforceable; escalations have no context.
- **Without Invariant 2**, "ship this with a quick fixup amend" silently collapses the revision sequence — fine until you need to bisect a regression to a specific revision.
- **Without Invariant 3**, "main agent fixed a small thing in Phase 10" silently ships unreviewed code under a green gate.

All three failure modes have happened in practice. The invariants exist because the failures happened, not because they sound rigorous.
