# Completeness Declarations

Each role declares what it actually covered. Producers declare; consumers (downstream roles + the ship-ready gate) check. A missing or false declaration is itself a finding.

The point: every handoff (spec → AC → test → code → verify → review) has a hidden completeness invariant (e.g., "all references swept", "all consumer surfaces walked"). Declarations make those invariants explicit so they can be audited.

## Smith — submission completeness

Embedded in Smith's submission schema (`subagents/smith.md`). Filled at every revision.

```yaml
completeness_declaration:
  symbols_renamed_or_removed:
    - symbol:               <name>
      refs_grep_command:    <exact command run>
      refs_found:           <integer>
      refs_touched:         <integer>
      all_touched:          true | false
      skipped_with_reason:  [<ref_location: reason>]   # only if all_touched == false

  ac_branches_tested:
    - ac:        AC1
      branches:  [happy, empty, error]                  # which branches the AC requires
      tests_covering:
        happy:   [<test names>]
        empty:   [<test names>]
        error:   [<test names>]
      gaps:      []                                     # any branch in `branches` with no test

  consumer_surface_swept:
    # For renamed types / changed schemas: enumerate consumption sites
    # and confirm they handle the new shape.
    - changed_artifact:      <type or schema name>
      consumption_sites:     [<file:line>]
      cast_or_deserialization_sites_checked: [<file:line>]
      verified_consistent:   true | false
```

**Rule:** filling honestly with `false` or non-empty `gaps` is **acceptable** — that surfaces a real open item for main agent / critic triage. Filling **falsely** (claiming `all_touched: true` when stale refs remain) will be caught by critics at Phase 6+ and counts as a blocker.

## Scout — verification completeness

Embedded in Scout's per-round output (`subagents/scout.md`).

```yaml
completeness_declaration:
  ac_items_verified: [AC1, AC2, ...]                    # every AC must appear
  ac_branches_verified:
    AC1: [happy, empty, error]
    AC2: [happy]
  emit_consume_chains_walked:
    - emit_site:    <file:line>
      consume_sites: [<file:line>, ...]
      cast_or_deserialization_sites_checked: [<file:line>, ...]
      end_to_end_verified:  true | false
```

**Rule:** Scout audits Smith's declaration as part of its own verification. If Smith's `consumer_surface_swept` claims `verified_consistent: true` but Scout's chain walk reaches a consumer Smith did not list → blocker against Smith.

## Hawk — review completeness

Embedded in Hawk's per-round output (`subagents/hawk.md`).

```yaml
completeness_declaration:
  files_reviewed_in_full: [<paths>]
  symbols_renamed_grep_verified:
    - symbol:               <name>
      hawk_grep_count:      <integer>
      matches_smith_declaration: true | false
  negative_path_branches_audited:
    - location: <file:line>
      branch:   <empty | null | error>
      finding_id_if_any:    <id or null>
```

**Rule:** Hawk independently greps for renamed symbols. If `matches_smith_declaration: false`, Smith's declaration was wrong → blocker against Smith.

## Cross-cutting rules

- Declarations are part of the ship-ready gate input. The gate (Phase 8) requires every Smith submission and every critic round to carry a non-degraded `completeness_declaration`.
- A "degraded" declaration is one where the producer left a field blank or marked it `unknown`. That is itself a finding — not a defect, but a gap that must be triaged before ship-ready.
- Three different roles missing three different completeness checks is a structural problem, not three individual lapses — these schemas exist to make all three explicit and verifiable.
