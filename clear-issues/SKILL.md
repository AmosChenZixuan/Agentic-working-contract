---
name: clear-issues
description: Use when the user explicitly invokes /clear-issues or says "clear the backlog". Orchestrates the full issue lifecycle from investigation through close. Explicitly invoked only — not triggered by general conversation about issues.
---

# Clear Issues

## Overview

Main agent orchestrates the full lifecycle of resolving GitHub issues. Each phase has explicit owners. User can override any decision with explicit instruction at any time.

**Core principle:** Issues are investigated before dispatched, implemented in isolated worktrees, reviewed by a separate agent, and only merged after CI passes. No shortcuts.

## When to Use

Explicitly invoke this skill when the user says:
- `/clear-issues`
- "clear the backlog"

Do NOT trigger on casual discussion about issues or questions about issue status.

---

## Phase 1: Grill

**Owner:** Main agent

### 1a. Triage — epics and layers

Before picking issues to grill, classify the open issues:

- **Epic issues** (contain a sub-issues checklist) are tracking containers — never grill or dispatch them. They close when all sub-issues close.
- **Sub-issues** have `Parent:`, `Layer:`, and `Depends on:` metadata in their body. Read these fields.
- **Standalone issues** (no parent/layer metadata) are treated as Layer 1 with no dependencies.

Build a dependency map from the metadata. An issue is **eligible** when:
- All issues in its `Depends on:` list are closed (merged to target branch), AND
- It is not an epic

Pick up to **3** eligible issues to grill in parallel. Prefer lower layers first — don't grill Layer 2 while Layer 1 issues in the same epic are still open.

### 1b. Grill

1. Run `/grill-me` on up to 3 eligible issues **in parallel**.
2. **For each issue**, as soon as grill-me signals ready AND checklist passes, dispatch immediately — do NOT wait for other in-flight grilling to complete:
   - [ ] Root cause / goal identified
   - [ ] Files/modules touched known
   - [ ] Test strategy defined
   - [ ] No open dependencies (verified against dependency map)
   - [ ] Dependency signals checked in issue comments first
3. Build file-scope map across grilled issues to flag module overlap before dispatching.
4. If a dependency is still open → comment on the blocked issue, skip dispatch, pick up next eligible issue.

**Implementer questions queue:** While main agent is mid-grill, implementer questions (from previously dispatched issues) queue up. Answer them after the current grill batch completes — unless a question reveals a dependency that would change which issues to grill. In that case, resolve the dependency signal immediately.

**Scope discovery:** If implementer discovers issue has multiple concerns → it stops, comments on issue, asks main agent. Do NOT broaden scope unilaterally.

**Implementer idle time:** Implementers are created in Phase 2, after grill clears for their issue. While waiting for their issue to be grilled, the implementer is not created yet — there is no idle time to waste. Questions from newly dispatched implementers are answered normally during subsequent grill cycles.

---

## Phase 2: Dispatch

**Owner:** Main agent

1. Create git worktree for each implementer (branch from the current working branch). Use `superpowers:using-git-worktrees` skill. Confirm the base branch with user if not obvious.
2. Read `dispatch-template.md` and dispatch implementer using the **Implementer Dispatch Template**.
3. Track issue state in task list:
   - `pending` → `grilling` → `dispatched` → `implement` → `review` → `CI` → `merged` → `closed`
4. Dispatch immediately per-issue — see Phase 1b for the per-issue dispatch rule.

---

## Phase 3: Implement

**Owner:** Implementer subagent (runs in worktree)

Follow the dispatch template exactly. Key rules:

1. **Write failing test first** — `/test-driven-development` skill mandates this. Test must fail before fix.
2. **Simplify synchronously** — wait for `/simplify` results, apply all findings, commit before PR.
3. **Scope discipline** — if scope discovery occurs, stop and ask. Do not broaden unilaterally.
4. **Notify** — after PR is up, notify main agent with PR URL.

---

## Phase 4: Review

**Owner:** Reviewer subagent

Read `dispatch-template.md`. When dispatched to review, use the **Reviewer Dispatch Template**.

**Scope adherence check:** The reviewer MUST verify the PR actually and only does what the issue scoped. If the implementer reverted the change just to pass review — the PR is obviously not ready. The fix must address the root cause, not merely make tests pass by removing the broken behavior.

---

## Phase 5: CI & Merge

**Owner:** Main agent

1. **Verify CI green yourself** — do not trust the reviewer's "ready" signal alone. Check the PR's CI status directly (`gh run list` or PR status badge). If CI is still running → **poll every 30s** (up to 10 min) until result. If CI is failed or skipped:
   - **Comment on PR**: "CI failed/skipped — holding merge. Dispatching fix."
   - **Dispatch implementer** to the same worktree to fix the failing tests.
   - Do NOT merge until CI is green.
2. Squash merge to the agreed branch.
3. Delete branch immediately after merge
4. Delete worktree after merge
5. Monitor CI post-merge:
   - If CI fails → dispatch implementer for fix, same workflow
   - Do not close issue until CI is green on the target branch

**CI waiting mechanism:** The main agent polls, not the reviewer. Reviewer signals "CI not green" and returns. Main agent owns the wait and dispatches implementer to fix if CI fails. Reviewer is freed up to handle other PRs.

**Conflict handling:** If target branch moves while PR is open → main agent detects conflict, dispatches implementer to rebase. Clean rebase → merge. Conflict-resolved changes → reviewer sanity pass before merge. After any rebase, immediately run `git diff` against the expected state to verify all intended changes survived — do not assume the merge or rebase was correct.

---

## Phase 6: Close

**Owner:** Main agent

1. After PR merged and CI green on the target branch:
   - **Verify the fix exists in master code before closing** — run verification commands to confirm the code state matches what the issue asked for (e.g., if issue says "remove package X", verify `grep package.json` shows it's gone; if issue says "replace lib A with lib B", verify no imports of A remain). CI passing proves tests run. Code inspection proves the fix exists.
   - Comment on issue with summary of changes and CI status
   - Close issue
2. **Epic auto-close:** After closing a sub-issue, check its parent epic. If all sub-issues in the epic's checklist are closed → close the epic with a summary comment.
3. **Unblock next layer:** After closing a sub-issue, check if any blocked issues now have all dependencies met. If so, they become eligible for the next grill cycle — pick them up immediately rather than waiting for the user to re-invoke.
4. If regression surfaces later:
   - File new issue, run full workflow from scratch
   - Post-mortem goes to MEMORY.md (not the closed issue)

---

## State Machine

```
IDLE ─► TRIAGE (classify epics, build dependency map)
              │
              ├──► EPIC (tracking only — closes when all sub-issues close)
              │
              └──► GRILLING (up to 3 eligible issues in parallel)
                        │         │
                        │         └───(per issue: grill-me ready + checklist passes + no blocker)
                        │                   │
                        │                   ▼
                        │              DISPATCHED ─► IMPLEMENT ─► REVIEW ─► CI ─► MERGED ─► CLOSED
                        │         (worktree+implementer)    (fresh impl on fix rounds)
                        |                                                       │
                        |                                              unblocks next layer
                        │
                        └──► BLOCKED (dependency still open)
                                 (comment added, re-evaluated after each close)
```


---

## User Interrupt Handling

- **Explicit instruction always wins** — "handle issue #X" overrides any queue
- **"Drop everything"** — main agent pauses current grill cycle, handles new issue immediately
- **Normal new request** — queued, answered after current phase completes

---

## Key Decisions (always apply)

1. **Verify CI green yourself** — check CI status directly before merge. "PR ready" means reviewed AND CI green. If CI is failed/pending/skipped, do not merge regardless of review signal.
2. Fresh implementer on fix rounds — no path dependency contamination
3. Scope adherence — see Phase 4. Reviewer verifies PR only does what the issue scoped; root cause must be fixed, not symptoms gamed to pass tests.
4. `git rebase --skip` is a red-line — never skip commits during rebase without first verifying what the skipped commit contains and getting explicit confirmation that skipping is safe. Skipping drops commits permanently.
5. Never close an issue based on CI passing or PR merge alone — always verify the actual code state in the target branch matches the fix the issue requested. If CI is green but the code doesn't match, the fix was lost (e.g., during a rebase) and must be re-applied.
6. `/grill-me` is primary signal; main agent checklist is the gate — grill-me declares first, checklist applied after before dispatching.
7. **Always confirm merge target and auto-merge** — before merging, explicitly confirm which branch to merge to (default to the current working branch, not master) and whether to auto-merge after CI passes or wait for user confirmation. Never assume.
