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

## Declared Regressions

[Tests this milestone is expected to break. Must be declared here
BEFORE implementation runs. Cannot be added after the fact.
Each must name a specific resolving milestone.]

- [ ] `[test/surface]` will regress because [reason]. Resolved by: [M#].

If none: "None."

## Open Questions

[The orchestrator resolves these before launching implementation.]
```
