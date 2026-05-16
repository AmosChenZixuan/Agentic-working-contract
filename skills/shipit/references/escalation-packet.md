# Escalation Packet

When a finding cannot be resolved within the critique loop, main agent produces an escalation packet for the user. User is the final arbiter in v1.

## Trigger types

1. **Per-finding deadlock** — `pushback_count` reached 3 with `status: open`
2. **Scout↔Hawk conflict** — Scout and Hawk make incompatible demands on the same code
3. **Regression spiral** — 2 consecutive Smith revisions without blocker-count strictly decreasing
4. **AC-defect escalation** — Smith pushes back claiming the AC is wrong (not subject to pushback budget)

## Packet schema

```yaml
escalation_type: per-finding | scout-hawk-conflict | spiral | ac-defect
feature:         <feature name>
branch:          <branch or worktree path>
spec_link:       docs/specs/YYYY-MM-DD-<slug>/spec.md
ac_link:         <inline or path>
final_revision:  N            # highest revision N completed

# For per-finding:
finding:         <full finding object including complete history>

# For scout-hawk-conflict:
finding_scout:   <full Scout finding>
finding_hawk:    <full Hawk finding>
contradiction:   <one paragraph: what Scout demands vs what Hawk demands, and why they can't both be satisfied>

# For spiral:
revision_log:
  - revision: N
    blocker_count_open: X
    diff_summary: <1 sentence>
  - revision: N+1
    blocker_count_open: Y
    diff_summary: <1 sentence>

# For ac-defect:
smith_argument: <Smith's argument that AC is wrong>
relevant_ac:    [AC1, AC2]

# Always:
current_state:
  blockers_open:      integer
  blockers_fixed:     integer
  blockers_acked:     integer
  blockers_escalated: integer
  advisories_open:    integer

ask_user:
  - <specific question or decision needed>
  - <e.g., "Accept escalated blocker hawk-safety-X as ship-ready exception?">
  - <e.g., "Override Hawk and ship as-is? Audit trail preserved.">
  - <e.g., "Revise AC3 to allow the behavior Smith implemented?">
```

## User's response options

User picks one per escalated item. Each maps to a `user_resolution` value that the ship-ready gate reads (see `finding-schema.md` and SKILL.md Phase 8). "Ship anyway, fix later" is not a permitted option — `accept_escalated` is its only legitimate audit-trailed form.

| Option label | `user_resolution` value | Effect on finding | Loop effect |
|---|---|---|---|
| **Accept escalated** | `accept_escalated` | `status: acked` (ship-ready exception, audit-trailed) | none |
| **Override critic** | `override_critic` | `status: acked` (force-close, audit-trailed) | none |
| **Side with critic** | `side_with_critic` | `status: open`, `pushback_count: 0` | Smith re-dispatched to fix; on critic re-verify pass, `status: fixed` |
| **Revise AC** | `revise_ac` | finding re-evaluated against new AC | Phase 5 restart (Smith re-dispatch with revised AC) |
| **Revise spec** | `revise_spec` | finding re-evaluated against new spec | Phase 3 restart (spec rewrite) |
| **Abort** | `abort` | finding frozen as-is | shipit terminates; branch left for manual handling |

Main agent **never** decides escalations unilaterally in v1. Main agent packages context; user arbitrates. Main agent records the user's choice into `finding.user_resolution`, then re-runs the ship-ready gate.

## Output format to user

Render the packet as a clearly-labeled section in the chat, not buried in tool output. Example header:

```
═══ ESCALATION: per-finding deadlock ═══
Feature: <name>
Branch:  <branch>
Finding: <id>
Severity: blocker
Pushback count: 3/3 (exhausted)

[Full finding + history]

Decisions required:
  1. <option>
  2. <option>
  3. <option>
```
