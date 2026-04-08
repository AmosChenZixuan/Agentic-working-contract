---
name: clear-issues
description: Use when the user explicitly invokes /clear-issues. This skill orchestrates the full issue lifecycle; grill → dispatch (worktree) → implement (TDD + simplify) → review → CI → merge → close. Explicitly invoked only — not triggered by general conversation about issues.
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

## The Workflow

```
  IDLE → GRILLING → DISPATCHED → IMPLEMENT → REVIEW → CI → MERGED → CLOSED
                        ↑            ↑                   │
                        │            │              needs fix (from REVIEW)
                        │            └────────────────────┘
                        └──── CI fail ──────────────────┘
```

---

## Phase 1: Grill

**Owner:** Main agent

1. Pull open issues. Pick up to **3** most clearly scoped issues to grill in parallel.
2. Run `/grill-me` on each issue.
   - `/grill-me` declares when ready
   - Main agent applies checklist before dispatching:
     - [ ] Root cause identified
     - [ ] Files/modules touched known
     - [ ] Test strategy defined
     - [ ] No open dependencies
     - [ ] Dependency signals checked in issue comments first
3. Check issue comments for dependency signals before grilling.
4. Build file-scope map across grilled issues to flag module overlap before dispatching.
5. If issue depends on a dispatched issue → comment dependency on issue, skip dispatch, grill next.

**Implementer questions queue** while main agent is mid-grill. Answer after grill phase completes.

**Scope discovery:** If implementer discovers issue has multiple concerns → it stops, comments on issue, asks main agent. Do NOT broaden scope unilaterally.

---

## Phase 2: Dispatch

**Owner:** Main agent

1. Create git worktree for each implementer (branch from current master). Use `superpowers:using-git-worktrees` skill.
2. Read `dispatch-template.md` and dispatch implementer using the **Implementer Dispatch Template**.
3. Track issue state in task list:
   - `pending` → `grilling` → `dispatched` → `implement` → `review` → `CI` → `merged` → `closed`
4. Dispatch as issues clear `/grill-me` — don't wait for all in-flight issues to clear.

**Rule:** Parallel dispatch for independent issues. Each PR merges when its own CI is green and review is approved.

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

---

## Phase 5: CI & Merge

**Owner:** Main agent

1. **Wait for CI before merge** — "PR ready" means reviewed, not mergeable
2. Squash merge to master
3. Delete branch immediately after merge
4. Delete worktree after merge
5. Monitor CI post-merge:
   - If CI fails → dispatch implementer for fix, same workflow
   - Do not close issue until CI is green on master

**Conflict handling:** If master moves while PR is open → main agent detects conflict, dispatches implementer to rebase. Clean rebase → merge. Conflict-resolved changes → reviewer sanity pass before merge.

---

## Phase 6: Close

**Owner:** Main agent

1. After PR merged and CI green on master:
   - Comment on issue with summary of changes and CI status
   - Close issue
2. If regression surfaces later:
   - File new issue, run full workflow from scratch
   - Post-mortem goes to MEMORY.md (not the closed issue)

---

## State Machine

```
IDLE
  │ pull issues, pick up to 3, run /grill-me
  ▼
GRILLING (1-3 issues in parallel)
  │ skill signals ready + checklist passes
  │ implementer questions queue here
  ▼
DISPATCHED (worktree created, implementer running)
  ▼
IMPLEMENT (TDD → fix → simplify → commit → PR submitted)
  │ PR submitted
  ▼
REVIEW
  │ needs fix → fresh implementer → back to IMPLEMENT
  │ ready → CI
  ▼
CI (waiting for checks)
  │ fail → dispatch implementer fix
  │ pass → MERGE
  ▼
MERGED (branch deleted, worktree cleaned)
  ▼
CLOSED (issue closed, MEMORY.md updated if needed)
```

---

## User Interrupt Handling

- **Explicit instruction always wins** — "handle issue #X" overrides any queue
- **"Drop everything"** — main agent pauses current grill cycle, handles new issue immediately
- **Normal new request** — queued, answered after current phase completes

---

## Key Decisions (always apply)

1. Wait for CI before merge — never merge on "PR ready" alone
2. Independent PRs merge as ready — no gate on other in-flight issues
3. Implementer stops and asks on scope discovery — don't broaden unilaterally
4. Fresh reviewer on each review round — no stale context
5. Simplify runs synchronously before PR submission — never submit dirty
6. Reviewer re-reviews post-simplify code — never review pre-simplify state
7. Reviewers dispatched as PRs are ready — don't wait for all PRs
8. Main agent monitors CI post-merge — owns all regressions
9. Fresh implementer on fix rounds — no path dependency contamination
10. Implementer notifies when PR updated — no polling
11. Dispatch immediately after issue clears grill — don't wait
12. Implementer questions queue during grill phase — main agent answers after grill completes
13. Main agent creates worktree before dispatch, cleans after merge
14. Branch deleted immediately after merge
15. Regression → new issue with full workflow — never silently patch
16. Post-mortem in MEMORY.md — not on closed issue
17. TDD runs for all issue types — skill mandate, always
18. Conflict → main agent dispatches implementer to rebase
19. Clean rebase → old approval stands; conflict-resolved diff → reviewer sanity pass
20. `/grill-me` declares done + main agent checklist (both required)
21. User requests queue during in-flight workflow — unless "drop everything"
22. Use EXACT dispatch templates from `dispatch-template.md` — do not ad-lib prompts
