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

Spawn subagent with `subagents/smith.md` as instructions. Pass: spec, AC, plan, design decisions log, current branch / worktree path.

### 6. Dispatch Scout + Hawk in parallel

After Smith submits. Spawn Scout with `subagents/scout.md` (pass: AC, Smith's diff, Smith's tests, feature type for tool selection). Spawn Hawk with `subagents/hawk.md` (pass: Smith's diff, surrounding code context). Do not pass design rationale to Scout — Scout stays black-box.

### 7. Critique loop

Collect findings. Route blockers to Smith. Smith revises. Scout and Hawk re-verify. Per-finding pushback ≤3 re-raises per id. Track spiral. Escalate any of: (a) per-finding 3-pushback exhaustion, (b) Scout↔Hawk conflict on same code, (c) regression spiral.

### 8. Convergence or escalation

- All blockers `fixed` or critic-accepted-justification → ship-ready
- Any blocker `escalated` → produce packet per `references/escalation-packet.md`, ask user. User decides per-finding: accept escalated, override, revise AC, revise spec, or abort.

### 9. Ship-ready handoff

Output to user:

- PR diff and PR description draft
- AC verification report (from Scout)
- Code review summary (from Hawk)
- Finding log (closed + escalated)
- Escalation packet (if any)
- Branch / worktree path

User merges manually. shipit never runs `gh pr merge` or `git push`.

### 10. Post-ship knowledge cleanup

Prefer `neat-freak`. Update docs, memory, CLAUDE.md if affected.

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
