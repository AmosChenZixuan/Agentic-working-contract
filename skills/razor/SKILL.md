---
name: razor
disable-model-invocation: true
description: >
  Strip a design down to the essential, before any code is written. Use whenever
  a feature, approach, architecture, or spec is being proposed or debated and not
  yet built — especially when you sense scope creep, or the user asks "is this
  overkill / over-engineered / too complex", or reaches for a heavy solution.
  razor reconstructs what the user *actually* needs (rarely what they first
  describe), derives the smallest design that meets it, and names everything
  beyond that as over-design. Pre-implementation only: it guards the design, it
  does not review written code.
---

# Razor

The right design is the *least* that fully meets the user's **actual** need —
nothing added for needs that don't exist yet. Two cuts: find the need, then fit
the solution to it.

**1. Find the real need.** What people ask for is a solution they already reached
for, not the need underneath it — and what they *say* often isn't what they
*think*. The stated request, the imagined solution, and the actual need are three
different things. Reconstruct the need from first principles: who is this for, and
what are they truly trying to do, right now. You can't cut against a target you
haven't found — if it's unclear, ask.

**2. Cut the solution to fit (Occam).** The right design is the fewest moving
parts that meet that need; every extra layer, option, abstraction, or guard is
cost — to build, read, test, break, and change — unless a *present* need pays for
it. To find the fewest, stop at the first rung that holds:

- Does it need to exist at all? → no: drop it (YAGNI).
- Does something already solve it — this codebase, the stdlib, a platform feature,
  an installed dependency? → use it, don't build it.
- Can it be one obvious thing instead of a system? → make it that.

Lazy about the solution, never about understanding the problem.

## Process

1. Reconstruct the true need (cut 1); state it in one line.
2. Derive the minimal design — start from the dumbest thing that works, add only
   what the need forces (cut 2).
3. Diff the proposal against it. Everything extra is a candidate cut; name the red
   flag (YAGNI, premature abstraction, over-defense, reinvention, speculative
   option).
4. Don't under-cut. Re-read the need and put back anything it actually requires.
   Narrow ≠ fragile — never cut real validation, error handling, security, or
   accessibility.

## Output

Use this template, and keep it short — a scalpel, not an essay:

```
## True need
<one line — what the user actually needs, not what they asked for>

## Minimal design
<the narrowest approach that meets it>

## Cut as over-design
- <thing> — <red flag + why no present need justifies it>

## Keep (don't over-cut)
<anything that looks cuttable but is genuinely needed, so it isn't dropped later.
Omit this section if nothing qualifies.>
```

If the design is already minimal, say so and stop.
