# BRIEF.md Template

The briefing agent writes `project-notes/<slug>/BRIEF.md`.

A brief should describe ONE focused change. If a milestone contains
multiple independent changes (e.g., "fix bugs 1-5"), split it into
separate milestones — one per change. The orchestrate pipeline handles
sequencing. Batching independent items into one brief causes shallow
work on each.

**Scope validation — the briefing agent must check before writing:**

A milestone is too broad if its acceptance criteria require multiple
independent root-cause diagnoses. One milestone = one diagnosis + one
fix + one verification. If the source issue combines multiple distinct
defects (e.g., "symbol canonicalization AND circular ownership AND
constructor ownership"), each defect is a separate milestone even if
they were filed as one issue. Split before briefing, not during
implementation.

Signs that a milestone needs splitting:
- The acceptance criteria list independent symptoms with different root
  causes
- The implementation approach has multiple "diagnose X, then diagnose Y"
  steps that could fail independently
- Fixing one criterion does not necessarily help or inform fixing another
- The original issue spans multiple components or subsystems

When splitting, the briefing agent should note the split in the first
brief's Open Questions section so the orchestrator creates the remaining
milestones.

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

[Key steps from PLAN.md. Enough to guide, not a second plan.

If a step implies multiple distinct responsibilities (e.g., "parse
input, validate, transform, and write output"), break it into separate
steps. Each step should map to roughly one function. If a single step
cannot be described without "and" or "then" joining unrelated concerns,
it is too broad and the implementation will produce an overly long
function.]

1. ...

## Constraints

- Design decisions that apply here
- Forbidden approaches
- Dependencies on other milestones
- Files or APIs that must NOT be modified
- Naming conventions to follow — note patterns in the existing code
  (e.g., `resolve_X` for lookups, `emit_X` for output). If the
  briefing agent notices inconsistencies in the files this milestone
  touches (e.g., mix of `get_` and `fetch_` for the same operation),
  flag them here so the implementation agent picks one and the audit
  can verify

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

## Abstraction Context

[The briefing agent must read the modules this milestone touches and
their immediate neighbors, then inventory the abstraction landscape.
This is not optional — abstraction problems caught after implementation
are expensive; catching them in the brief is cheap.

List each existing abstraction (function, trait, protocol, module
boundary) that the milestone will use, extend, or work alongside.
For each, note its health:]

### Existing abstractions

- `path::to::abstraction` — [what it does, who calls it]
  - Health: [complete | incomplete | leaky | duplicated by X]
  - [If incomplete]: callers do [extra work] around it because [reason]
  - [If leaky]: callers depend on [internal detail] instead of interface
  - [If duplicated]: also implemented as `path::to::other`

### Abstraction requirements for this milestone

[Based on the inventory above, state what the implementation must do
with respect to abstractions:]

- **Must use**: [existing abstraction] — do not reimplement
- **Must extend**: [existing abstraction] to cover [new case]
- **Must not duplicate**: [concept] already has [abstraction]
- **Must fix**: [incomplete/leaky abstraction] as part of this milestone
  (only if the milestone's scope includes it)

[If no relevant abstractions exist and this milestone introduces new
behavior, state whether a new abstraction is warranted or whether
inline code is appropriate. The default is inline — only create an
abstraction when there are or will be multiple call sites.]

## Declared Regressions

[Tests this milestone is expected to break. Must be declared here
BEFORE implementation runs. Cannot be added after the fact.
Each must name a specific resolving milestone.]

- [ ] `[test/surface]` will regress because [reason]. Resolved by: [M#].

If none: "None."

## Open Questions

[The orchestrator resolves these before launching implementation.]
```
