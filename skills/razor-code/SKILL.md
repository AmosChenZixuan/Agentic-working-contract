---
name: razor-code
argument-hint: "[<omit for current diff, or a PR #>]"
description: >
  Cut cruft from code that already exists. Use after an implementation, diff, or
  PR is written and you want it leaner — when reviewing changes, or asked to
  "simplify this", "is this over-engineered / too much code", "trim the tests",
  "clean up the comments", or "review my changes for bloat". razor-code dispatches
  fresh-eyes reviewer subagents and aggregates what to remove across four cuts:
  over-engineering & over-defense, needless complexity & duplication, dead-weight
  comments, and low-value tests — never touching what earns its place, never
  accepting a behavior change. Post-implementation only: it cuts written code. To
  guard a design before it's built, use razor instead.
---

# Razor-code

Every line is a liability — to read, test, break, and maintain. The best code is
the *least* that does the job correctly and safely. razor-code reviews code that
already exists and names what to cut: complexity that buys nothing, defense
against the impossible, comments that lie or restate, tests that test nothing.

Two non-negotiables:

- **Behavior is sacred.** A clean cut removes code whose absence *cannot*
  change any output, side effect, or contract on any path — edge and error
  included — and that invariance must hold **in place**: established by this
  file's own logic, its types, or a schema constraint. Invariance that holds
  only because another module maintains it is not invariance but a dependency;
  cutting there moves a safety property away from the code that needs it, and a
  later edit *there* breaks *here* with the suite still green. Whatever you
  can't establish in place is behavior-affecting: name the invariant as the
  risk, route it to sign-off, never present it as safe.
- **Fresh eyes, not the author's.** Reviewers see the code, never the reasoning
  behind it, so cuts aren't rationalized away (the dispatch below is what keeps
  them cold).

## The four cuts

Each reviewer applies one cut.

**1. Over-engineering & over-defense.** Walk the ladder; stop at the first rung
that holds:

- Does it need to exist at all? → no: drop it (YAGNI).
- Already solved — this codebase, the stdlib, a platform feature, an installed
  dependency? → reuse it, don't reinvent it.
- Can it be one obvious thing instead of a system? → make it that.

Flag: speculative options and config, wrapper layers over a native feature,
hand-rolled logic that reimplements something the stdlib, a platform feature,
or an installed dependency already ships, abstractions with a single caller,
infrastructure built ahead of a real need, and guards for states that cannot
occur.

**2. Complexity & duplication.** Flatten needless nesting and dead branches;
consolidate duplicated or near-duplicated logic; delete dead code and unused
params, exports, and imports. This cut restructures, it does not redesign.
Clarity over brevity: never trade a readable block for a dense one-liner or a
nested ternary.

**3. Comments.** Cut comments that restate what the code already says; that have
rotted out of sync with it; or that lean on external markers that rot —
TODO/FIXME, ticket and issue IDs, doc links, "see PR #…", changelog narration.
Keep the comments that explain *why*: a non-obvious rationale, a subtle
invariant, a deliberate trade-off.

**4. Tests.** Cut tests that earn nothing: boilerplate scaffolding, duplicate
cases over the same path, trivial getter/setter checks, and tests coupled to
implementation detail that break on a safe refactor rather than on a behavior
change. Before calling a test a duplicate, confirm the test you credit as
"already covering it" hits the **same branch and inputs** — calling the same
function is not the same path. A sibling using different inputs (distinct
timestamps where this one pins a collision) isn't covering it; this test is the
sole verifier of its path — keep it. Keep — and flag if *missing* — the tests that catch real regressions:
behavior, contracts, edge and error paths. Weigh **cost** too: a test whose
runtime or setup is disproportionate to what it covers is a liability even when
green. If it's also low-value → cut it. If it's the only thing covering a real
behavior → keep the coverage but flag the cost to be reduced (hoist a
shared/module-scoped fixture, parametrize, shrink the payload); never delete the
sole verifier of a behavior to save time.

## Process

1. **Scope.** Default to the current change (the diff); narrow it if the user
   names a cut ("just the tests") or a path.
2. **Dispatch reviewers.** Size the dispatch to the change: a small diff → one
   subagent carrying all applicable cuts; a large or multi-area diff → one
   subagent per cut, in parallel. Skip a cut whose code isn't present (no test
   changes → no test reviewer). Each subagent starts cold and cannot see this
   skill, so put everything it needs in its prompt: the diff and surrounding
   code — but **not** the design rationale, whose absence is what keeps its eyes
   fresh — plus, for every cut it owns, that cut's brief above, the behavior
   rule, and the finding schema. A reviewer returns findings only; it never
   edits.
3. **Aggregate.** Collect every finding. Dedupe overlaps (over-engineering and
   complexity will collide), reconcile conflicts, and split into safe cuts vs.
   behavior-affecting flags. Re-check the keep guard: never drop trust-boundary
   validation, data-loss handling, error handling, security, accessibility, or a
   test that covers real behavior.
4. **Report** in the template below.

## Finding schema

Reviewers return findings in this shape, no prose:

```
cut:           over-engineering | complexity | comments | tests
location:      file:line  (a range or several sites is fine)
remove:        <what to cut>
why:           <why it earns nothing, one clause>
behavior_risk: none | <the output, side effect, or contract it could change>
```

`behavior_risk: none` → a safe cut. Anything else → behavior-affecting, routed
to sign-off.

## Output

Keep it short — a scalpel, not an essay. A safe cut or a keep is a claim, not
an argument: state it in one line and move on. Spend sentences only where a
human has to make a judgment call — that's Behavior-affecting, and nowhere
else.

```
## Safe to cut
- <file:line>: <what> → <replacement, or "nothing">. <why it earns nothing, one clause>.

## Behavior-affecting (needs sign-off)
- <file:line>: <what looks cuttable> — <why it isn't safe: name the invariant,
  the dependency, or the risk. Full reasoning belongs here, not above.>

## Keep (don't over-cut)
- <file:line>: <what> — <why it earns its place, one clause>. Omit this
  section if nothing qualifies.

## Gaps
<missing tests for real behavior or edge paths — the one place razor-code adds
rather than removes — plus necessary-but-expensive tests whose cost should be
cut without losing coverage. Omit if none.>

net: -<N> lines cut, <M> flagged for sign-off.
```

The `net:` line is always present, even at zero — it's the one number a
reviewer can trust without reading the rest.

If the code is already lean, say so and stop.
