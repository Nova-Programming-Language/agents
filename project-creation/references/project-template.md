# PROJECT.md Template

```md
# Project Backbone

## Summary

- problem:
- intended outcome:

## Current Lifecycle Stage

- requirements | architecture | implementation_plan | implementation | validation

## Required Links

- PRD:
- Architecture docs:
- Implementation plan:

Use `N/A` with a reason when a link does not apply.

## Acceptance Criteria

- AC1:
- AC2:

## Completion Semantics

- `scaffolded`: declarations or structure exist; behavior is incomplete
- `implemented-but-not-exposed`: all committed branches work internally but
  are not public
- `exposed`: the independently usable public boundary and all promised
  branches are implemented and tested
- `release-verified`: all acceptance criteria, completion-matrix cells, package
  checks, and final regression gates pass

Only `release-verified` may be reported as fully complete.

## Nova Delivery Context

### Affected Nova Surfaces

- `nova run`:
- `nova check`:
- `nova repl`:
- `nova test`:
- `nova doc`:
- `nova-test-runner`:
- interpreter-only execution:
- compiled backend execution:
- docs/examples:

Mark `yes`, `no`, or `N/A` and add short notes when useful.

### Authoritative Sources

- spec docs:
- architecture docs:
- metadata contract docs:
- `prelude.novi` impact:
- runtime ABI or native-library impact:

### Observable Semantic Change

- user-visible behavior changing:
- behavior that must remain unchanged:

### Missing-Data / Invalid-State Behavior

- what must fail loudly:
- where fallback is forbidden:

### Shared Semantic Families

- constructs that should share a path:
- constructs that truly diverge:

## Review Gates

- Architecture draft critique: pending | ready_for_human_review | revise_before_human_review | N/A
- Architecture human feedback: pending | received | acted_on | N/A
- Implementation plan critique: pending | ready_for_human_review | revise_before_human_review | N/A
- Implementation plan human feedback: pending | received | acted_on | N/A

## Scope

- In scope:
- Out of scope:

## Owning Components

- frontend
- interpreter
- codegen-c
- CLI
- test runner
- docs

## Locked Decisions

- decision:

## Open Questions / Blockers

- blocker:
- question:

## Validation Proof Summary

- exact commands:
- cross-surface checks:
- negative checks:
```
