# PLAN.md Template

```md
# Implementation Plan

## Objective

- what this plan delivers:

## Project Scope

- affected command surfaces (profile §Command surfaces):
- authoritative docs:
- project-specific impact areas (profile §Delivery checks):
- behavior that must fail loudly if invalid or missing:

## Milestones

### M1

- scope:
- independently usable outcome:
- owning components:
- expected proof:
- expected commit shape:

### M2

- scope:
- independently usable outcome:
- owning components:
- expected proof:
- expected commit shape:

Split a milestone only per the orchestrate brief template's "Scope
validation" and "Size by risk and reward" rules.

## Completion Matrix

Required for broad public contracts. Each applicable cell must name automated
proof or an explicit blocker; compilation and declarations do not count.

| Contract item | Success/failure | Engines/surfaces | Lifecycle | Packaging | State |
|---|---|---|---|---|---|
| [item] | [tests] | [tests] | [tests] | [tests] | scaffolded / implemented-but-not-exposed / exposed / release-verified |

## Cross-Command Impact

- [one line per command surface listed in profile §Command surfaces]

Mark only the paths that actually apply.

## Validation Plan

- acceptance criteria mapping:
- exact commands:
- cross-surface checks:
- negative validation:
- build or environment prerequisites:
- manual checks:
- milestone gates (tiers per the regressions skill, "When to Run What"):

## AI Critique

- verdict: pending | ready_for_human_review | revise_before_human_review
- major findings:
- revisions applied:

## Dependencies / Blockers

- dependency:
- blocker:

## Notes

- sequencing assumptions:
- related constructs that should stay unified:
- true divergence points:
```

Size and split milestones only per
`../../orchestrate/references/brief-template.md`, "Scope validation" and "Size
by risk and reward".
