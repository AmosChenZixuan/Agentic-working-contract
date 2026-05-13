---
name: agent-scope
description: First phase of the agent development chain. Use this skill when starting any non-trivial feature, fix, or change — before planning or writing code. Covers scoping, clarification, and writing a spec. Triggers on "let's scope this", "help me think through X", "what's the best approach for Y", or any request where the requirement is ambiguous or the approach is undecided. Always runs before agent-plan.
---

# agent-scope

You are in phase 1 of 3. Your job is to turn a vague requirement into an unambiguous spec that `agent-plan` can slice into work.

**Chain:** `agent-scope` → `agent-plan` → `agent-code`

---

## Phase 1: Understand the context

Before asking anything, read what's already there:
- Relevant code, existing patterns, recent commits (`git log --oneline -10`)
- Any existing spec or design doc

Come prepared. Don't ask the user for things you can find yourself.

---

## Phase 2: Clarify (one question at a time)

Ask only what you can't resolve by reading the code. One question per message — not a list. Wait for the answer before asking the next one.

You're done clarifying when you can answer all of these:
- What problem does this solve, and for whom?
- What does success look like — concretely?
- What is explicitly out of scope?
- Are there constraints (performance, compatibility, existing interfaces to preserve)?

---

## Phase 3: Propose approaches

Present 2–3 approaches with tradeoffs. Be concrete — name the files, patterns, and interfaces involved. Let the user choose or redirect before you write the spec.

---

## Phase 4: Write the spec

Once the approach is agreed, write the spec to `docs/specs/YYYY-MM-DD-<slug>.md`.

```markdown
# <Feature name>

## Problem
<What this solves and why it matters>

## Approach
<The chosen approach — enough detail to plan from>

## Scope
### In scope
- <concrete deliverable>

### Out of scope
- <explicit exclusion>

## Acceptance criteria
- <observable, testable condition>
- <observable, testable condition>

## Open questions
- <anything unresolved>
```

**Hard gate:** The spec must have at least one concrete acceptance criterion before handing off. "Works correctly" is not an acceptance criterion.

---

## Phase 5: Hand off

After the user approves the spec (or makes corrections), say:

> "Spec is at `docs/specs/<filename>`. Ready to plan — invoke `agent-plan` with that file."

Do not start planning. Do not write code. Your job ends here.
