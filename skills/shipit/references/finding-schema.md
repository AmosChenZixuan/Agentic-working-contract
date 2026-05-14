# Finding Schema

Every critic finding (from Scout or Hawk) MUST use this exact schema. No prose reviews accepted.

## Fields

```yaml
- id:               string   # stable, immutable, unique per finding
  raised_by:        Scout | Hawk
  severity:         blocker | advisory
  category:         correctness | safety | spec-compliance | perf | style | regression
  location:         file:line  OR  ac-item-id
  issue:            string   # ≤2 sentences, what's wrong
  suggested_fix:    string   # ≤2 sentences; optional for advisory
  rationale:        string   # ≤1 sentence, why this matters
  pushback_count:   integer  # 0..3, incremented on each re-raise
  status:           open | fixed | acked | escalated
  user_resolution:  null | accept_escalated | override_critic | side_with_critic | revise_ac | revise_spec | abort
                    # ONLY set when status == escalated and user has arbitrated.
                    # Required for the ship-ready gate to pass on escalated findings.
  history:          []       # ordered log of Smith↔critic exchanges
  differs_from:     []       # optional; list of prior finding ids when new finding lands within 10 lines of a prior one
```

## Severity semantics

- **blocker** — must be resolved (fixed, escalated→user-resolved, or critic-accepted justification) before ship-ready
- **advisory** — Smith acks and decides; no loop; may be deferred to a follow-up PR

### Severity lock

Severity is determined by behavior, not by cost-of-fix or PR scope. Critics MUST NOT downgrade `blocker` to `advisory` because:

- Smith argues "out of scope this PR"
- Smith argues "we'll fix it in a follow-up"
- Smith argues the fix is large or risky

These are all escalation grounds (to main agent → user), not downgrade grounds. The audit trail records the dispute either way. Allowing scope-based downgrades is how shipped PRs accumulate silent known bugs.

The following are always `blocker`:
- Data silently dropped on any path (happy, empty, error)
- AC unmet under exercise on any required branch
- Regression in code adjacent to the diff (non-feature behavior change)
- Safety violation at any trust boundary

## Status transitions

```
open → fixed       (Smith's revision addresses finding AND critic re-verifies pass)
open → acked       (advisory: Smith acknowledges and decides;
                    OR blocker: critic accepts Smith's justification — see Severity lock above for what does NOT qualify)
open → escalated   (pushback_count reached 3; OR Scout↔Hawk conflict on same code; OR regression spiral active)
escalated → fixed  (user resolution `side_with_critic`, `revise_ac`, or `revise_spec` produced a passing re-verify)
escalated → acked  (user resolution `accept_escalated` or `override_critic` — audit-trailed exception)
```

`fixed` and `acked` are terminal. `escalated` is terminal **only after** `user_resolution` is set; otherwise it blocks ship-ready.

## ID discipline

- Critic assigns the id when first raising. Format: `<critic>-<category>-<short-slug>`. Examples: `hawk-safety-null-deref-user`, `scout-spec-compliance-AC3-missing-test`.
- Same finding re-raised → exact same id, increment `pushback_count`.
- New finding caused by Smith's fix → NEW id, `pushback_count: 0`.
- "Near-but-different" finding within 10 lines of a prior one → new id BUT MUST declare `differs_from: [prior_id]` with a one-sentence root-cause distinction.
- ID-laundering (cosmetic rename to reset counter) is forbidden. The audit trail in the escalation packet exposes the pattern.

## `history` entries

Each entry records one exchange:

```yaml
- by:      Smith | Scout | Hawk
  round:   integer    # critique round number
  action:  raise | fix | justify | re-raise | accept | escalate
  content: string     # the actual exchange text — code snippet, justification, or counter-argument
```

`history` survives across revisions. Escalation packets include the full history for user arbitration.

## Example finding

```yaml
- id:              hawk-safety-unchecked-body-size
  raised_by:       Hawk
  severity:        blocker
  category:        safety
  location:        src/api/upload.ts:42
  issue:           Request body parsed without size limit; large payload can OOM the worker.
  suggested_fix:   Add `express.json({ limit: '1mb' })` middleware before this handler.
  rationale:       Boundary safety — untrusted input.
  pushback_count:  1
  status:          open
  history:
    - by: Hawk
      round: 0
      action: raise
      content: Request body parsed without size limit at src/api/upload.ts:42. Add 1mb cap.
    - by: Smith
      round: 0
      action: justify
      content: Upstream reverse proxy already caps at 2mb (see infra/nginx.conf:18). Adding express-level cap is redundant.
    - by: Hawk
      round: 1
      action: re-raise
      content: Proxy cap is infra-level, not enforced in test environments where this code runs directly. App-level cap is defense-in-depth, not redundancy.
```
