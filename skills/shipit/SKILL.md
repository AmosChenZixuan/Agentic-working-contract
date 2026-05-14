---
name: shipit
description: Use when delivering a single PR-scoped feature end-to-end with multi-critic review loops. Triggers on "/shipit", "ship this feature", or any request for full-cycle feature delivery from design through ship-ready PR with structured critique.
---

# shipit

Orchestrator skill. Main agent owns design, AC, planning, and arbitration. Spawns three named subagents per feature:

- **Smith** — white-box implementer who forges the feature end-to-end (design interpretation → tests → code → refactor), keeping full feature context with no mid-flight compaction.
- **Scout** — black-box AC verifier who reconnoiters from outside; exercises the feature via the appropriate tool (playwright MCP for web UI, curl/shell/test-runner otherwise) without reading design rationale.
- **Hawk** — white-box code reviewer with sharp eyes for correctness, safety, regression, perf, and style.

Scout and Hawk run in parallel post-coding and feed structured findings back to Smith in a bounded critique loop. Output is a ship-ready PR — never auto-merged.

## Core principles

- **Context-boundary over task-boundary.** Smith holds full feature context end-to-end. No mid-flight compaction. Eliminates inter-agent re-derivation cost.
- **Single PR scope.** Smith's initial loaded context ≤60k tokens. Heuristic: if diff likely >800 LOC or touches >8 files, split feature into multiple PRs upstream during planning.
- **Parallel critics, structured findings only.** Scout + Hawk run in parallel after Smith's first submission. All findings use the schema in `references/finding-schema.md`. No prose reviews.
- **Per-finding pushback budget = 3.** Critic re-raises same finding id at most 3 times. Then that finding escalates. Review loop continues on other findings until no blockers remain.
- **Spiral detector.** Two consecutive Smith revisions without blocker-count strictly decreasing → escalate as regression spiral.
- **Never auto-ship.** End state is a ship-ready PR. User merges manually.
- **Never on main.** Smith refuses to work on `main` or `master`. Worktree preferred, feature branch fallback.

## Role boundaries

Each role has explicit allowed inputs, allowed outputs, and forbidden actions. Boundary violations are an orchestrator bug, not a judgment call. The point is to prevent role drift under time pressure or when an upstream agent has terminated.

| Role | Allowed outputs | Forbidden actions |
|---|---|---|
| **Main agent** | spec, AC, plan, dispatch decisions, escalation packets, ship-ready handoff | edit code in the critique loop; modify findings' severity or status; resolve escalations without explicit user input |
| **Smith** | code, tests, submission schema, justification text in finding `history` | modify AC; modify findings' severity/category; mark own finding `fixed` (only critic re-verify can) |
| **Scout** | AC verification report, findings | edit code; accept a scope-based downgrade ("out-of-scope this PR") — that is always an escalation to main agent, never `acked`; negotiate AC text |
| **Hawk** | code-review findings | edit code; accept a scope-based downgrade; flag AC compliance (that is Scout's surface) |

**The escalation principle:** when an agent encounters a situation outside its allowed outputs, it escalates to main agent. It does not improvise. Main agent's own forbidden actions force it to escalate to the user instead of self-resolving.

**Critique-loop re-dispatch rule:** if Smith's subagent has terminated and a new blocker requires code change, main agent re-dispatches a fresh Smith subagent with the state packet (spec, AC, findings, prior diff, revision_count). Main agent never edits code directly to "close out" a finding.

## Phase log (state machine)

shipit is a state machine, not a sequence of suggestions. Main agent maintains a `phase_log` for the entire run. Each phase has explicit **entry conditions** (which must hold to start) and **exit conditions** (which must hold to leave). A phase's exit condition is the next phase's entry condition.

Phases CANNOT be skipped, reordered, or marked complete without their exit conditions actually holding. The ship-ready gate (Phase 8) consumes `phase_log` directly: any missing or non-exited phase fails the gate.

```yaml
phase_log:
  - phase: <id>                   # 0a, 0b, 1..10, plus 5.5 (verification)
    entered_at: <ts>
    entry_conditions_met:  [<conditions actually checked>]
    exited_at: <ts or null>       # null while in progress
    exit_conditions_met:   [<conditions actually checked>]
    artifacts: [<paths or refs produced>]
    notes: <optional>
```

Worktree path is persistent state on the log: Phase 0a records `worktree_path`, and every subsequent phase asserts `pwd == worktree_path` (or uses absolute paths) before any file operation. Path drift across tool calls is the most common silent-failure source — Bash's working directory persists across calls, but Read/Write cache can mislead. Re-pin every Bash call with an explicit `cd <worktree_path> &&` or use absolute paths throughout.

## Phases

### 0. Bootstrap

Two checks at session start:

**0a. Branch / worktree check.** Inspect git state:

- If `superpowers:using-git-worktrees` is available, prefer it.
- Otherwise:
  ```bash
  git rev-parse --abbrev-ref HEAD
  git worktree list
  ```
  - If current branch is `main` or `master` → **block**. Prompt user:
    > "shipit refuses to work on `<branch>`. Create a worktree (preferred) or feature branch? [worktree / branch / abort]"
    - `worktree` → `git worktree add ../<repo>-<slug> -b <feature-slug>` then `cd` to it.
    - `branch` → `git checkout -b <feature-slug>`.
    - `abort` → stop.
  - If already on a non-main branch but it's a long-lived shared branch (`develop`, `staging`, `release/*`) → warn and confirm.

**0b. Skill / MCP probe.** Inspect available skill list and configured MCP servers. Output the per-phase fallback tier (preferred / local fallback / degraded) to the user. For any `degraded` phase, prompt user once: install the preferred skill or proceed degraded. Defer to user; never auto-install. See `references/skill-fallback-matrix.md`.

### 1. Background gather

Read repo: `git log --oneline -10`, existing patterns, related code, any prior spec under `docs/specs/`. Do not ask the user what you can read yourself.

### 2. Clarify intent

Prefer `superpowers:brainstorming`. Local fallback `grill-me`. Degrade: ask one question per turn until you can answer — problem, success criteria, out-of-scope, constraints.

### 3. Write spec + AC

Use `references/spec-template.md`. Write to `docs/specs/YYYY-MM-DD-<slug>.md`. Hard gate: at least one observable, testable AC. "Works correctly" is not an AC.

### 4. Plan

Prefer `superpowers:writing-plans`. Otherwise inline slices with closing boundary tests per slice. Estimate Smith's initial token load. If >60k or diff/file heuristic exceeded, split feature into multiple PRs and stop.

### 5. Dispatch Smith

Spawn subagent with `subagents/smith.md` as instructions. Pass: spec, AC, plan, design decisions log, **absolute path to worktree** (not a relative path; Smith must operate inside this path).

**Exit conditions:** Smith returns a submission schema. Smith's submission is an assertion at this point — Phase 5.5 verifies it before it counts as fact.

### 5.5. Submission verification (mechanical, non-skippable)

Smith's submission is a self-report. Main agent verifies it against repo state before Phase 6 dispatch. Subagent output cannot be trusted on its face — Smith may hallucinate code that was never written, tests that don't exist, or files that were never touched.

Verification steps, all run from the worktree path:

```bash
cd <worktree_path>
git status --porcelain                       # any uncommitted changes?
git diff --name-only HEAD~<n>..HEAD          # which files actually changed across Smith's commits
git log --oneline HEAD~<n>..HEAD             # how many commits Smith actually made
<project's test runner> -v                   # actual test count and pass/fail
```

Cross-check against Smith's submission:

| Smith claims | Verify against |
|---|---|
| `files_touched: [a, b, c]` | `git diff --name-only` matches exactly (no missing files, no surprise files) |
| `tests_added: [...]` | each test file exists; test runner discovers them; count matches |
| test pass count | test runner output matches |
| `revision: N`, branch N commits | `git log` shows N commits authored in this session |
| `completeness_declaration.symbols_renamed_or_removed[*].all_touched: true` | independent grep confirms no stale references |

If ANY check fails, submission is `rejected_unverified`. Main agent's only allowed action is **re-dispatch Smith** with the verification failure passed in as part of the state packet. Main agent does NOT patch the gap itself. (Same forbidden action as the critique-loop re-dispatch rule — see Role boundaries.)

After 2 consecutive `rejected_unverified` results from the same Smith chain, escalate to user: Smith may be hallucinating or the task may be malformed. Do not loop indefinitely.

**Exit conditions:** all submission claims verified against repo state; `verification_report` artifact written to `phase_log`.

### 6. Dispatch Scout + Hawk in parallel

Entry condition: Phase 5.5 verification passed. If Phase 5.5 has no exit record, Phase 6 cannot start.

Spawn Scout with `subagents/scout.md` (pass: AC, Smith's verified diff, Smith's tests, feature type for tool selection). Spawn Hawk with `subagents/hawk.md` (pass: Smith's verified diff, surrounding code context). Do not pass design rationale to Scout — Scout stays black-box.

### 7. Critique loop

Collect findings. Route blockers to Smith. Smith revises. Scout and Hawk re-verify. Per-finding pushback ≤3 re-raises per id. Track spiral. Escalate any of: (a) per-finding 3-pushback exhaustion, (b) Scout↔Hawk conflict on same code, (c) regression spiral.

If Smith's subagent has terminated between revisions, re-dispatch a fresh Smith with the state packet. Main agent never patches the code itself to close a finding (see Role boundaries above).

### 8. Ship-ready gate

`ship_ready` is a computed state, not a label Smith claims. Main agent verifies the gate mechanically before producing the handoff. The gate is:

```
ship_ready :=
      every finding.status ∈ {fixed, acked}
  AND no finding has status == open
  AND no finding has status == escalated without a recorded user_resolution
        ∈ {accept_escalated, override_critic, side_with_critic→fixed, revise_ac→fixed, revise_spec→fixed}
  AND every required phase (0a, 0b, 1..5.5, 6, 7, 10) has an exit record in phase_log
  AND every submission's completeness_declaration is present and non-degraded
        (see references/finding-schema.md and subagents/smith.md)
  AND every Smith submission has a verification_report artifact (Phase 5.5)
```

If the gate fails, main agent does NOT ship. Options:

- Blocker `open` → route back into Phase 7 (re-dispatch Smith if needed).
- Blocker `escalated` with no user resolution → produce packet per `references/escalation-packet.md`. User picks one of the response options. Main agent records the chosen `user_resolution` in the finding, then re-checks the gate. **"Ship anyway, file follow-up issue" is not a permitted resolution** — it must be `accept_escalated` (which is a deliberate audit-trailed override), not silent dismissal.
- Completeness declaration missing → that itself is a finding raised against the submitting agent; loop continues.
- Phase 10 not yet run → run it (it is part of the gate, not optional cleanup).
- Phase exit record missing → that phase has not actually run, regardless of how complete the artifacts look; resume from there.

### 9. Ship-ready handoff

Entry condition: Phase 8 ship-ready gate returned true.

**Step 9.1 — Create draft PR.** From the worktree (or feature branch) path, push and open a draft PR:

```bash
cd <worktree_path>
git push -u origin <branch>                  # if not yet pushed
gh pr create --draft \
  --title  "<title following project commit/PR convention>" \
  --body   "<body assembled from Scout AC report + Hawk summary + finding log>"
```

The PR is opened as `--draft` to make "ship-ready, not shipped" explicit in the GitHub UI. Title should follow the project's existing PR convention — inspect recent merged PRs (`gh pr list --state merged --limit 10`) to infer the pattern.

**Step 9.2 — Hand off to user.** Output to chat:

- PR URL (from Step 9.1)
- AC verification report (from Scout)
- Code review summary (from Hawk)
- Finding log (closed + escalated, with `user_resolution` for any escalated)
- Escalation packet (if any)
- Branch / worktree path
- `phase_log` summary
- Suggested commit / merge strategy (default: leave to user; squash vs merge-commit is project-dependent)

**Step 9.3 — Forbidden actions in Phase 9.**

- `gh pr merge` — never. Ship-ready is not shipped.
- `git push --force` — never in this phase (only allowed if explicitly resolving an escalation that called for it, and only with user confirmation).
- Marking the PR as "ready for review" (un-drafting) — that is the user's call, not shipit's.

### 10. Post-ship knowledge cleanup

Prefer `neat-freak`. Update docs, memory, CLAUDE.md if affected.

This phase is part of the ship-ready gate, not optional polish. The handoff at Phase 9 must record that Phase 10 has run (or that the user explicitly waived it via `--skip-cleanup`).

## Reference files

- `subagents/smith.md` — Smith's prompt (white-box implementer)
- `subagents/scout.md` — Scout's prompt (black-box AC verifier)
- `subagents/hawk.md` — Hawk's prompt (white-box reviewer)
- `references/finding-schema.md` — structured finding fields, severity, id discipline
- `references/escalation-packet.md` — escalation triggers and format
- `references/spec-template.md` — spec + AC template
- `references/skill-fallback-matrix.md` — per-phase fallback chain and install commands

## Hard rules

- Never auto-merge. Output is ship-ready, not shipped.
- Never work on `main` / `master`. Worktree preferred, feature branch fallback.
- Smith must not skip Scout or Hawk. Both run on every feature, even trivial ones.
- All critic findings use the structured schema. No prose reviews accepted.
- Re-raise must reuse the exact finding id. New issue caused by Smith's fix = new id with `pushback_count: 0`.
- Escalation packet always includes full finding `history`, not just final state.
- User is sole arbiter of escalations in v1. Main agent packages context; main agent does not decide.
- Subagent submissions are unverified assertions until Phase 5.5 mechanically verifies them against repo state. Trust no `tests_added`, `files_touched`, or pass count without `git diff` and test-runner confirmation.
- Phases are a state machine. Every required phase must have an exit record in `phase_log` before the ship-ready gate can pass. No skipping, no reordering.
- Worktree path is persistent state. Every Bash call uses absolute paths or explicit `cd <worktree_path> &&`. Re-pin every call; do not rely on shell `cwd` persistence.
- No `git commit --amend` inside the critique loop. Each Smith revision is a new commit. Git audit trail is as important as findings audit trail. Final squash is the user's choice at merge time, not Smith's during review.
