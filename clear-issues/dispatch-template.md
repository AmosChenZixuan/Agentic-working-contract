# Dispatch Templates

Use these EXACT templates when dispatching implementer or reviewer subagents.

---

## Implementer Dispatch Template

Fill in all placeholders before dispatching.

```
You are fixing GitHub issue #<NUMBER>: <title>

**Issue structure:**
- Epic: #<EPIC_NUMBER> (or "standalone" if no parent)
- Layer: <N> of <TOTAL>
- Depends on: <merged dependency PRs, or "none">

**Context from grill phase:**
- Root cause: <root_cause>
- Files to touch: <file_list>
- Test strategy: <test_strategy>
- Platform: <backend/web — what subsystem>
- Project context: <any relevant conventions from CLAUDE.md>

**Steps — follow in order:**

1. **Read first** — read the files above to understand existing patterns
2. **Clean stale tests** — remove or fix broken tests, not weaken assertions
3. **Run /test-driven-development** — write a failing test that reproduces the bug, then implement fix. REQUIRED for all issue types.
4. **Run /simplify** synchronously — wait for results, apply all findings, commit fixes. REQUIRED before PR.
5. **Submit PR** — push branch, open PR against master
6. **Notify main agent** — reply with: PR URL, summary of changes, test results

**PR message format:** `fix | <platform> | #<NUMBER> <short description>` (e.g. `fix | backend | #123 remove auth token storage`)

**Constraints:**
- Do NOT change files outside scope listed above
- Do NOT merge to master
- Do NOT close or merge the PR
- If you discover the issue has multiple unrelated concerns → stop, comment on the issue, ask main agent. Do NOT broaden scope.

**Return:** PR URL and summary of what you did.
```

---

## Reviewer Dispatch Template

Fill in all placeholders before dispatching.

```
Review PR for issue #<NUMBER>

PR URL: <url>
Files changed: <list>

**Review steps:**

1. **Scope adherence check** — verify the PR only contains changes scoped to the issue. If the implementer reverted the change just to pass review, the PR is NOT ready. The root cause must be fixed, not the symptom.
2. Run `/code-review` skill on the PR
3. Evaluate findings:
   - If issues found → dispatch FRESH implementer to same worktree with specific fix requests. Do NOT review the fix yourself — let the implementer fix and push.
   - If clean → signal main agent: "PR #<NUMBER> ready to merge"

**Approval criteria:**
- All `/code-review` findings addressed
- Test coverage adequate
- No new tech debt introduced
- Simplify findings were applied
- PR only contains changes scoped to the issue — if the implementer reverted the change just to pass review, the PR is NOT ready; the root cause must be fixed, not the symptom

**Rules:**
- Fresh reviewer on each review round — do NOT reuse a reviewer who already reviewed this PR
- Approval after clean rebase stands (fast-forward, no conflict resolution changes)
- If rebase introduced conflict resolution changes → sanity pass required before merge

**Return:** "PR #<NUMBER> ready" OR "PR #<NUMBER> needs fix — <specific issues>"
```
