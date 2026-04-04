# Design Document Rules

**Template:** `assets/design-doc-template.md` — start here for new design docs. Fill in the `{{ placeholders }}` and delete sections that don't apply. For individual Architecture Decisions, use `assets/adr-template.md` instead.

**Audience:** Engineers (human and AI) who need to understand architectural decisions, system contracts, and the reasoning behind non-obvious choices. They already have the code — they're reading this to understand *why*.

## What belongs

- **Decisions with rationale** — what was chosen, what the alternative was, and why this choice was made. Especially valuable when the code doesn't reveal the tradeoff.
- **System contracts** — what one component promises to another (API shapes, DB schema as a structural contract, data flow between layers)
- **Non-goals** — explicit statements of what the system intentionally doesn't do
- **Open questions** — unresolved design choices, with status (resolved/unresolved)

## What doesn't belong

- UI component internals — agents read the components
- Exhaustive column or field listings — show structure, not every attribute
- Route annotations that restate what the route name already says
- Implementation swaps (e.g. library A replaced by library B) unless the switch changed the architectural contract
- Anything the code makes obvious

## Schema and API sections

Show the *structure and purpose* of each table or endpoint, not a full column dump. Include a column only if:
- Its name or type is non-obvious
- It affects behavior in another part of the system
- It encodes a decision worth documenting (e.g. a nullable FK that represents an optional relationship)

## Decisions table

Every significant architectural choice belongs in a decisions table with three columns: Decision, Choice, Rationale. The rationale column is the most important — it should explain the tradeoff, not just restate the choice.

## Keeping it readable

A design doc that's updated after every PR becomes a changelog, not a design document. Update it when:
- A new structural contract is established
- A significant decision is made or reversed
- A new component is added to the architecture diagram
- An open question is resolved

Do not update it to reflect routine feature additions, new UI components, or refactors that don't change the architecture.
