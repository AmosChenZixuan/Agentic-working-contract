---
name: write-like-me
description: >
  Match the user's own writing voice whenever Claude drafts text that will read
  as authored by the user — bug reports, PR/commit messages, GitHub/chat replies,
  SKILL.md or README prose, design-pivot notes, drafts. Terse, lowercase-first,
  no closing punctuation, CN/EN code-switching, reason-first pushback. Not for
  Claude's own conversational turn — only for content attributed to the user.
---

# Write Like Me

Patterns learned from the user's own messages. Match the pattern, never reuse
an example line verbatim — write new text that follows the same rule.

## 1. Fragment over sentence

**Problem:** default drafting writes full subject+verb+period sentences even
when a fragment carries the same information.
**Before:** "I've finished the task, and there is one test still failing."
**After:** "done, 1 test failing"

## 2. Lowercase-first

**Problem:** capitalizing every sentence start signals a formality the user
doesn't use. Casing carries no meaning here — don't "fix" it.
**Before:** "Approved. Please merge."
**After:** "approve to merge"

## 3. No closing punctuation

**Problem:** a trailing period or "!" reads as over-finished. A sentence just
stops; "?" appears only for a real question.
**Before:** "The complexity is O(n) time and O(1) space."
**After:** "O(n) time O(1) space"

## 4. Commas chain, periods don't

**Problem:** splitting related clauses into separate sentences over-formalizes
what's really one run-on thought.
**Before:** "This is done. There's one AC left. I'll merge after review."
**After:** "done, 1 AC left, merge after review"

## 5. Code-switch CN/EN by function

**Words to watch:** English carries code, algorithm/complexity notation, and
terse status words (done/next/merge/approve). Chinese carries meta-reasoning,
judgment calls, and pushback where nuance matters more than the English
term's precision.
**Problem:** translating a bilingual thought into one language "for
consistency" erases the actual signal — which language a clause is in tells
you whether it's a status update or a judgment call.
**Before:** "I don't think this argument holds up, it's overcomplicated."
**After:** "这不成立，想复杂了"

## 6. Reason-first pushback, never hedged

**Words to watch:** "wrong", "reject", "驳回", "no" — always immediately
followed by the concrete reason, never a bare rejection and never softened
with "I think maybe..." first.
**Problem:** hedging a correction buries the reason the reader actually needs.
**Before:** "I'm not sure this is the right approach — it might create some
coupling issues, but I could be wrong."
**After:** "no, this creates a dependency on X, breaks cross-platform"

## 7. Evidence over assertion

**Words to watch:** "why", "查清楚，不要靠猜" (don't guess, go verify).
**Problem:** accepting a claim at face value instead of asking for the chain
of proof behind it.
**Before:** "That should be fine, I'll trust that it works."
**After:** "why — show me it actually works"

## 8. Constraints stated up front

**Problem:** stating a process rule after the fact reads as a correction; the
user states it before the task starts instead.
**Before (after-the-fact):** "Actually, I wanted you to just discuss this,
not write code yet."
**After (up front):** "only discuss, no code here"

## Don't

- No closing summary, recap, or "let me know if..." — the message ends when
  the point is made.
- No full grammatical sentence where a fragment says the same thing.
- No apology padding before a correction.
- No forcing a bilingual draft into a single language.

## What not to flag

- **Formal, complete English** when it's text-being-edited (resume lines,
  product copy, docs meant for someone else) — that's a different register
  the user deliberately keeps polished, not a lapse to "fix."
- **A genuine full sentence** when the content actually needs the subordinate
  clause — don't chop a sentence that would lose meaning as a fragment.

## Self-check

Closing period on most lines, a wrap-up sentence, formal-memo tone → cut it
down. Soft-sounding rejection → sharpen it, put the reason right after.
