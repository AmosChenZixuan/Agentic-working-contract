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
6. **Write PR description** — MUST include:
   - **Change summary**: 2-4 bullets of what changed and why
   - **AC checklist**: Copy the issue's acceptance criteria, mark each `[x]` (done) or `[ ]` (NA). Required before review.
7. **Notify main agent** — reply with: PR URL, summary of changes, test results

**PR message format:** `fix | <platform> | #<NUMBER> <short description>`

**Constraints:**
- Do NOT change files outside scope
- Do NOT merge or close the PR
- **Do NOT force push** — create NEW commits for fixes; force push destroys review history
- If scope discovery reveals unrelated concerns → stop, comment on issue, ask main agent

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

1. **PR description check** — verify change summary and AC checklist are present. If missing → return "PR #<NUMBER> description missing required content".
2. Run `/review` skill on the PR
3. **Write ALL review comments into the PR** — for every issue found, post a GitHub review comment with file, line, problem, and suggested fix. Do NOT just return text.
4. **Verify CI is green** — check PR's CI directly. If still running → re-check once after a short wait. If failed/skipped → return "PR #<NUMBER> CI not green — <failed/skipped>".
5. **Scope + AC verification** — verify: (a) PR only contains changes scoped to the issue, (b) each AC item is satisfied. If implementer reverted the change just to pass review → NOT ready. Root cause must be fixed.
6. Evaluate:
   - Clean + CI green + all AC met → signal main agent: "PR #<NUMBER> ready to merge (CI green)"

**Approval criteria:**
- PR description has change summary and AC checklist
- All `/review` findings addressed
- Test coverage adequate
- No new tech debt introduced
- Simplify findings were applied
- All AC items satisfied and PR only contains scoped changes
- CI is green

**Rules:**
- **ALL review findings written as GitHub PR comments** — history must be visible in PR, not just agent sessions
- Approval after clean rebase stands; conflict resolution changes require sanity pass
- No force push in this workflow

**Return:** "PR #<NUMBER> ready to merge (CI green)" | "PR #<NUMBER> needs fix — <issues>" | "PR #<NUMBER> CI not green — <failed/skipped>"
```
