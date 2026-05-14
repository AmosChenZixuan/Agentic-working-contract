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
spec_link:       docs/specs/YYYY-MM-DD-<slug>.md
ac_link:         <inline or path>
revision_count:  N

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

User picks one per escalated item:

- **Accept escalated** — mark as `acked`, treat as ship-ready exception. Audit trail preserves the dispute.
- **Override critic** — force-close the finding, ship-ready. Audit trail preserves the override.
- **Side with critic** — Smith must fix. Pushback budget resets for this finding. Restart loop.
- **Revise AC** — main agent updates AC; affected slices restart from Phase 5 (Smith re-dispatch with new contract).
- **Revise spec** — full restart from Phase 3 (spec rewrite).
- **Abort** — stop. No ship. Branch left in current state for manual handling.

Main agent **never** decides escalations unilaterally in v1. Main agent packages context; user arbitrates.

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
