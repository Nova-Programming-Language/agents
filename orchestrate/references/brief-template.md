# BRIEF.md Template

The briefing agent writes `project-notes/<slug>/BRIEF.md`.

One brief = one focused change. Batching independent items causes shallow
work. The orchestrate pipeline handles sequencing.

**Scope validation** — check before writing: if acceptance criteria require
multiple independent root-cause diagnoses, split into separate milestones.
One milestone = one diagnosis + fix + verification. Signs of too-broad scope:
independent symptoms with different root causes, multiple "diagnose X then Y"
steps, fixing one criterion doesn't inform fixing another, issue spans
multiple components. Note splits in Open Questions for the orchestrator.

```md
# Task Brief: [milestone name]

## Objective

[One sentence: what this milestone achieves and why.]

## Acceptance Criteria

[Numbered, verifiable. Exact command, expected output, or observable behavior.]

1. ...

## Relevant Files

- `path/to/file` — [role: defines X / implements Y / tests Z]

## Implementation Approach

[Key steps from PLAN.md. Each step should map to roughly one function.
If a step needs "and" or "then" joining unrelated concerns, split it.
If the architecture doc has a matching Common Change Pattern, list the
co-required component updates here.]

1. ...

## Constraints

- Design decisions, forbidden approaches, dependencies, files not to modify
- Naming conventions — note existing patterns and flag inconsistencies

## Done When

- [ ] [command] produces [expected output]
- [ ] [behavior] is observable via [method]

## Behavioral Inventory (refactoring only)

[Required when moving, extracting, or consolidating code. List every
discrete behavior from the caller/user perspective. Cross-reference test
coverage: "covered by `test_name`" or "no test coverage".]

- [ ] [behavior] — in `path/to/old.ext` (~lines N-M)

## Abstraction Context

[Read architecture doc and neighboring modules. List existing abstractions
this milestone touches with health assessment.]

### Existing abstractions

- `path::to::abstraction` — [what, who calls it]
  - Health: [complete | incomplete | leaky | duplicated by X]

### Requirements for this milestone

- **Must use**: [abstraction] — do not reimplement
- **Must extend**: [abstraction] to cover [new case]
- **Must not duplicate**: [concept] already has [abstraction]
- **Must fix**: [abstraction] (only if in scope)

[Default is inline code — only abstract when multiple call sites exist.]

## Declared Regressions

[Must be declared BEFORE implementation. Each names a resolving milestone.]

- [ ] `[test]` will regress because [reason]. Resolved by: [M#].

If none: "None."

## Open Questions

[Orchestrator resolves before launching implementation.]
```
