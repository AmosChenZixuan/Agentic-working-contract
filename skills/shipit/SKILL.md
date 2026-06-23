---
name: shipit
description: Use when delivering one agent-ready unit of work end-to-end into a review-ready PR. Triggers on "/shipit", "ship this", "ship this issue/spec", or any request to take a single feature/bug/refactor from spec through a reviewed, review-ready pull request. Takes ONE agent-ready unit (a GitHub issue, a spec, or a clear request) and produces ONE non-draft PR a human can review and merge. The final stage of the AWC chain: grill-me → to-issues → shipit. Not for clearing a backlog (shipit handles one unit per run) and not for merging (the human owns merge + close).
---

# shipit

Take **one agent-ready unit** → ship **one review-ready PR**. The main agent plans, codes, and commits; it spawns reviewer subagents only. It never merges and never closes the issue — a human reviews the PR and owns merge + close.

The AWC chain is `grill-me → to-issues → shipit`. shipit expects the spec to already be agent-ready; if it isn't, shipit creates one inline (below) rather than refusing.

## Flow

### 1. Spec — get to agent-ready

A unit is **agent-ready** when it states: a single goal, scope (in / out), the files/modules touched, and acceptance criteria (AC) that are observable and testable. Read the input — a GitHub issue, a spec file, or the conversation.

If it isn't agent-ready, build it inline: invoke `grill-me` to clarify the gaps, draft the spec, then invoke `razor` to cut it to the smallest design that meets the real need.

Then normalize the AC — **whatever the source**, a ready issue needs this too. Each AC gets a stable id (`AC1`…), names its required branches (`happy`, plus `empty`/`error` where a negative path exists), and states an outcome **observable from outside the code** — a returned value, an HTTP response, a rendered screen, a CLI exit — so the blackbox reviewer can verify it in step 4. At least one AC; none may read "works correctly".

### 2. Workspace

Never work on main/master — refuse and prompt. Honor a workspace mode the user already stated; otherwise ask `[branch / worktree]`, never a silent default. `branch` → `git checkout -b <slug>` (slug from the goal) from the repo's main ref. `worktree` → `superpowers:using-git-worktrees` if available, else `git worktree add`. `cd` in, then write the resolved spec to `spec.md` here — the only state shipit keeps on disk; it becomes the PR body.

### 3. Plan

`razor` the approach (skip if step 1 already razored a fresh spec), then plan the work as TDD slices: for each AC branch, the failing test that pins it, then the minimal code to pass it. Keep the plan to the smallest set of slices the AC forces.

### 4. The loop — code, then review, until clean

Main holds full context end-to-end and writes all code. Reviewers are subagents and never edit; they return findings, main fixes. Stages run **sequentially** — whitebox fully clears before blackbox starts.

1. **Code** (main). Write the failing tests for the slice, then the minimal implementation, then run the full suite green (the test command comes from the repo — CI config or package scripts). Commit. Commits are **incremental and never amended** — every revision is a new commit, so the trail is fully traceable. A revision with no passing test run is not done.
2. **Whitebox** — invoke `razor-code` on the diff. It dispatches its own cold-eyes reviewers and returns findings. Apply safe cuts; any blocker → back to step 1.
3. **Blackbox** — the AC verifier (`subagents/blackbox.md`). It walks the **full AC checklist** and confirms each from outside, picking the method per AC: live smoke / visual via MCP + screenshots where the AC is about a rendered or interactive surface, otherwise curl / shell / the test runner / output inspection. It runs on every feature — any spec has at least one AC. Any blocker → back to step 1.

Any code change re-enters at whitebox (re-run whitebox, then blackbox) — scope each re-run to the **incremental diff since the last clear**, not the whole PR, to keep the loop cheap. The loop exits only when whitebox returns zero blockers **and** blackbox confirms every AC with zero blockers. No iteration cap and no mid-loop escalation — the human's gate is the PR review.

Whitebox (`razor-code`) reviews for leanness, not correctness — it holds behavior sacred and won't hunt bugs. Inside shipit, correctness is guarded by **blackbox's AC verification**: does the feature actually do what each AC says, on every required branch. Deeper adversarial bug-hunting is intentionally **post-shipit** — a human, or review agents, run against the review-ready PR. shipit's job is to produce a PR worth reviewing, not to be the last line of defense.

### 5. PR + handoff

Push, then `gh pr create` (**not** a draft — the reviewers already passed; title per repo convention). Body = the spec + a one-line whitebox summary + the blackbox AC report + the finding log; add `Closes #NN` if the unit was a GitHub issue. Post the **session reflection as a PR comment** (what was built, honest misses, anything the reviewer should look at hardest). Report `REVIEW-READY @ <url>`. The human reviews, merges, and closes.

## Skill preference

Prefer the installed skill per step; degrade inline if absent, and say so once. Never auto-install.

| Step | Prefer | Else |
|---|---|---|
| Clarify spec | `grill-me` / `superpowers:brainstorming` | inline Q&A |
| Cut design | `razor` | inline scope trim |
| Workspace | `superpowers:using-git-worktrees` | `git worktree add` / `git checkout -b` |
| TDD | `superpowers:test-driven-development` | inline test-first |
| Whitebox | `razor-code` | one cold reviewer subagent with the four cuts inlined |

## Lean finding

Blackbox returns findings in this shape (razor-code returns its own schema). No prose reviews.

```yaml
id:       <short-slug>
severity: blocker | advisory
issue:    <=2 sentences
fix:      <=2 sentences        # optional for advisory
status:   open | fixed | acked
```

Severity follows behavior, not cost-of-fix or PR scope: silently dropped data on any path, an AC unmet on a required branch, an adjacent regression, and a safety violation at a trust boundary are always `blocker`. A `blocker` loops until fixed. An `advisory` is acked and decided without looping.

## Hard rules

- One unit per run → one PR. No backlog walking, no dependency graphs.
- Never main/master. Never merge, never close the issue — output a review-ready PR; the human owns merge + close.
- No `git commit --amend`, no `git push --force`. Every revision is a new commit.
- Code is green (full suite passing) before any review runs.
- Both reviewers return zero blockers before the PR opens; blackbox verifies every AC.
- Main writes all code; reviewers only review. Reviewers never edit.
