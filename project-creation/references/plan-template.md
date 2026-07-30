# PLAN.md Template

```md
# Implementation Plan

## Objective

- what this plan delivers:

## Nova Scope

- affected Nova surfaces:
- authoritative docs:
- metadata / `prelude.novi` / runtime ABI impact:
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

Splits require an independently releasable/auditable outcome or measured
independent root cause. Time, file count, context size, and a failing test are
not split criteria.

## Completion Matrix

Required for broad public contracts. Each applicable cell must name automated
proof or an explicit blocker; compilation and declarations do not count.

| Contract item | Success/failure | Engines/surfaces | Lifecycle | Packaging | State |
|---|---|---|---|---|---|
| [item] | [tests] | [tests] | [tests] | [tests] | scaffolded / implemented-but-not-exposed / exposed / release-verified |

## Cross-Command Impact

- `nova run`
- `nova check`
- `nova repl`
- `nova test`
- `nova doc`
- `nova-test-runner`
- compiled backend paths

Mark only the paths that actually apply.

## Validation Plan

- acceptance criteria mapping:
- exact commands:
- cross-surface checks:
- negative validation:
- build or environment prerequisites:
- manual checks:
- milestone gates: targeted tests during implementation; owning suite once at
  milestone completion; canonical/full regression only at defined
  production-changing gates and final release

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
