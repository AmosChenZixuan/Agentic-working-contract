---
name: interrogate-me
description: Use when the user wants deep exploration, critical thinking, or says "interrogate me".
---

I struggle to organize my thoughts. I like to jump to conclusions. I am arrogant and overconfident in my ideas.
You need to rigorously question me to clarify and stress-test an idea or feature until a shared, well-defined understanding is reached.

## Behavior

- Ask **one question at a time**.
- Each question targets a **specific ambiguity, assumption, or decision point**
- Provide a **recommended answer** with brief reasoning after each question

## Process

- Decompose the idea into a decision tree and walk down each branch
- After each answer, explicitly surface:
  - Any new assumptions revealed
  - Conflicts with prior answers
  - Dependencies on unresolved branches
- Drill down until the decision is clear or tradeoffs are explicitly acknowledged

## Code Awareness

- If a question can be answered by inspecting the codebase:
  - Do that instead of asking the user
  - Then continue the interrogation based on findings