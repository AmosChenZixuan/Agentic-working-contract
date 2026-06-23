Parent: #NN — Epic title (omit if no epic)
Layer: N (omit if no epic)
Depends on: #NN, #NN (or "none")
Platform: backend | web | both (omit if N/A)

## Context
What exists today. Why this issue is needed — link the spec, doc, repro, or upstream signal that motivated it. Any constraints (perf budget, compatibility, design system) the implementer must respect.

## Goal
The single outcome this issue produces. One paragraph. State the observable behavior change, not the implementation path.

## Design
Type-specific. Use the sub-block that matches the issue type and delete the others.

### Feature
- **Schema** — SQL migrations, TypeScript types, or data shapes
- **API** — endpoint signatures (method, path, request body, response body, status codes)
- **UI** — component props, layout, states (loading, empty, error), and which existing component to extend
- **Modules** — function signatures for new code, with brief responsibility notes
- **Patterns to follow** — name real files in the repo that exemplify the approach

An agent reading only this section should be able to implement the work without consulting the spec.

### Bug
**Root cause** — the specific code path, race condition, missing check, or contract violation that causes the defect. Name the file and function/line where the bug originates. Quote the offending code if the location is non-obvious.

**Fix** — the change that resolves the root cause. Concrete enough for an agent to implement without further investigation. If multiple fix shapes are possible, name the chosen one and why.

If the issue is actually a behavior change rather than a true bug, this issue is misclassified — reconsider as a feature.

### Refactor
**Current state** — how the code works today. Name real files, functions, modules, and the patterns they follow. Quote the specific lines or call sites that need to change. Identify what is load-bearing and what is incidental.

**Target state** — how the code should look after. Concrete enough for an agent to implement:
- New file structure and module boundaries
- New or changed type definitions and function signatures
- The migration path — what gets renamed, moved, deprecated, or removed
- Patterns to follow, with file references

If mechanical vs behavioral, call it out. Mechanical refactors can be reviewed line-by-line; behavioral ones need a different eye.

If behavior must change, this issue is misclassified — split the behavior change into a separate issue.

## Files
Explicit list of files to create or modify, one per line. Mark (new) or (modify).

## Test strategy
How the work is verified:

- **Unit** — what to cover and where the test files live
- **Integration / e2e** — which flows to assert
- **Manual** — anything that needs human eyes (visual, a11y, latency)
- **Regression** — for bugs: the reproduction case, now expected to pass
- **Adjacent cases** — for bugs: what else could share the root cause
- **Structural assertion** — for refactors: a check that confirms the new shape
- **Performance check** — only if the refactor is perf-motivated

## Out of scope
What is explicitly NOT part of this issue. List related-but-deferred work to discourage bundling and to set reviewer expectations. Each deferred item is a separate issue.

## Acceptance
- [ ] [Objectively verifiable criterion — observable behavior, not "works correctly"]
- [ ] [Objectively verifiable criterion]
- [ ] [Tests added or updated per Test strategy]
