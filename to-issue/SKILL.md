---
name: to-issue
description: Convert conversations into GitHub issues. Use whenever the user invokes `/to-issue` — whether with a feature idea, a bug, a refactor point, or any finding worth capturing. This skill detects the issue type from context, fills the appropriate template, and auto-submits to the current repo. Also use proactively when the conversation surfaces bugs, design gaps, or deferred work that belongs in GitHub.
---

# to-issue

Convert any finding from conversation into a GitHub issue — fully structured, auto-submitted.

## Core concept

Two ways an issue gets created:

1. **User-led** — user says `/to-issue I want a new feature` and the agent helps structure it (user story, alternatives, additional context, grill-me analysis, etc.) until the template is full
2. **Agent-proposed** — during conversation, the agent notices a bug, refactor point, or deferred work and the user says `/to-issue` to capture it

In both cases: **template fills out → auto-submit → report issue link**.

## Step 1 — Detect repo

Use the current working directory as the repo. Run:

```
git remote get-url origin
```

Extract the GitHub owner/repo from the URL (e.g. `github.com/owner/repo.git`).

If no remote or ambiguous → ask the user explicitly.

## Step 2 — Determine issue type and infer title

Scan the conversation since the last `/to-issue` invocation (or from session start if first call) and determine:

**Type** (agent-infers from content):
- `Feature` — new functionality request
- `Bug` — bug report
- `Refactor` — refactoring opportunity
- `Docs` — documentation gap
- `Performance` — performance concern
- `Security` — security issue
- `Tech Debt` — technical debt item

**Title format**: `<type>: <concise description>`

Generate a concise, descriptive title from the conversation. For Scenario 1 (user-led), the title captures the user's intent. For Scenario 2 (agent-proposed), it captures the finding.

## Step 3 — Select and fill template

Load the template matching the detected type from `references/`. Fill every section possible from the conversation. Use best practices for each section (reproduce steps, acceptance criteria, etc.).

If a template section cannot be filled with current information — leave it blank or mark as `TBD` rather than fabricating.

Templates live in `references/`:
- `references/bug.md`
- `references/feature.md`
- `references/refactor.md`
- `references/docs.md`
- `references/performance.md`
- `references/security.md`
- `references/tech-debt.md`

If no type matches, use `feature.md` as the default.

## Step 4 — Determine readiness

A template is **ready to submit** when all required sections are filled with non-placeholder content.

If not ready → continue the conversation to gather what's missing. Do not submit until the template is filled.

## Step 5 — Submit via `gh`

Run:
```
gh issue create \
  --repo owner/repo \
  --title "<type>: <concise description>" \
  --body "$(cat <filled-template-as-markdown>)"
```

If `gh` is not installed or not authenticated → output the composed issue as markdown directly so the user can create it manually.

## Step 6 — Report

Return the issue URL to the user:
```
Created: <issue-url>
```
