# BRIEF.md Template

The briefing agent writes `project-notes/<slug>/BRIEF.md`.

One brief = one cohesive milestone outcome. Keep co-required branches,
engines, and lifecycle work together when they form one public contract.
Do not reduce the approved outcome merely to make a smaller brief.

**Scope validation** — split only when measurements establish independent root
causes or each result is independently auditable and releasable. Time, file
count, context size, component count, or one failing test is insufficient.
Never invent `a`/`b`, pilot, cleanup, or follow-up phases; propose a split to
the orchestrator and preserve the original acceptance criteria until approved.

```md
# Task Brief: [milestone name]

## Objective

[One sentence: what this milestone achieves and why.]

## Acceptance Criteria

[Numbered, verifiable. Exact command, expected output, or observable behavior.]

1. ...

For a broad public contract, reference its completion-matrix rows. Include
success/failure behavior, all promised engines/surfaces, lifecycle, and
packaging. A subset cannot satisfy the original criteria.

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
- [ ] no production placeholder, unconditional unsupported branch, empty
  success, ignored error, or promised-but-unreachable path remains

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
