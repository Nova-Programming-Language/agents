# BRIEF.md Template

The briefing agent writes `project-notes/<slug>/BRIEF.md`.

A brief should describe ONE focused change. If a milestone contains
multiple independent changes (e.g., "fix bugs 1-5"), split it into
separate milestones — one per change. The orchestrate pipeline handles
sequencing. Batching independent items into one brief causes shallow
work on each.

```md
# Task Brief: [milestone name]

## Objective

[One sentence: what this milestone achieves and why. Must describe a
single coherent change, not a batch.]

## Acceptance Criteria

[Numbered list from PROJECT.md, filtered to this milestone.
Each must be verifiable — exact command, expected output, or
observable behavior.]

1. ...

## Relevant Files

- `path/to/file` — [role: defines X / implements Y / tests Z]

## Implementation Approach

[Key steps from PLAN.md. Enough to guide, not a second plan.]

1. ...

## Constraints

- Design decisions that apply here
- Forbidden approaches
- Dependencies on other milestones
- Files or APIs that must NOT be modified

## Done When

- [ ] [command] produces [expected output]
- [ ] [behavior] is observable via [method]

## Behavioral Inventory (refactoring milestones only)

[Required when this milestone moves, extracts, or consolidates code.
Skip for pure additions or bug fixes.

The briefing agent must read the source being refactored and list every
discrete behavior, code path, or capability it provides. This inventory
becomes the contract: the implementation agent must account for every
item, and the audit verifies nothing was dropped.

Each entry is one behavior — not a function name or line range, but
what the code *does* from the caller's or user's perspective.]

- [ ] [behavior description] — currently in `path/to/old.ext` (lines ~N-M)
- [ ] ...

[If the old code has tests, cross-reference: "covered by `test_name`"
or "no existing test coverage". Untested behaviors are the ones most
likely to be silently dropped.]

## Declared Regressions

[Tests this milestone is expected to break. Must be declared here
BEFORE implementation runs. Cannot be added after the fact.
Each must name a specific resolving milestone.]

- [ ] `[test/surface]` will regress because [reason]. Resolved by: [M#].

If none: "None."

## Open Questions

[The orchestrator resolves these before launching implementation.]
```
