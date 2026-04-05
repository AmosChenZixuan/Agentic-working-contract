---
name: agent-scope
description: Use this skill when starting any non-trivial feature, fix, or change — before planning or writing code. Covers scoping, clarification, and writing a spec. Triggers on "let's scope this", "help me think through X", "what's the best approach for Y", or any request where the requirement is ambiguous or the approach is undecided. Always runs before agent-plan.
---

# agent-scope

You are in phase 1 of 3. Your job is to turn a vague requirement into an unambiguous spec that `agent-plan` can slice into work.

**Chain:** `agent-scope` → `agent-plan` → `agent-code`

---

## First Principles Thinking

The user may not know exactly what they want. Your job is to help them find it — not to guess.

- **When something is vague, ask.** Don't assume you understood. Don't fill in blanks with your best guess. Ask the one question that unlocks the rest.
- **When something is too big, offer simpler alternatives.** "Build a caching layer" might really be "I want API responses to stop timing out" — a proxy timeout config might solve it faster. Don't accept scope, offer solutions.
- **When something is too small to spec, say so.** If the request is already concrete and self-contained, skip the spec and offer to go straight to `agent-code`.
- **Don't accept "just do it" as a mandate.** If the request is ambiguous and the user pushes to proceed anyway, surface the ambiguity one more time before complying.

The goal is to build the right thing — not to build what's asked for.

---

## Phase 1: Understand the context

Before asking anything, read what's already there:
- Relevant code, existing patterns, recent commits (`git log --oneline -10`)
- Any existing spec or design doc

Come prepared. Don't ask the user for things you can find yourself.

---

## Phase 2: Clarify

Apply `interrogate-me` if available. Conduct one-at-a-time questioning to clear vagueness. Ask only what you can't resolve by reading the code.

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
