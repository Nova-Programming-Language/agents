# Semantic Family

Use this reference when two constructs look different locally but may belong to
the same semantic family at a higher level.

## Core Rule

Do not special-case a construct at a higher layer than necessary.

First identify the highest shared semantic layer. Share that layer across
related constructs. Introduce separate paths only at the first layer where
observable semantics diverge, and state what divergence requires it.

## Classification

Before adding a new path for construct `X`, compare it to the closest existing
construct `Y`:

1. Same semantics:
   Use one path.
2. Specialization or restricted form of the same family:
   Use a thin specialization over a shared core path.
3. Related but observably distinct:
   Share the lower semantic layer and split only where behavior first diverges.

Syntax differences alone do not justify a separate lowering, metadata shape, or
runtime path.

## Required Questions

1. What semantic family does `X` belong to?
2. What existing construct already covers that family?
3. Is `X` identical, a specialization, or a sibling construct?
4. At what exact layer do the semantics first diverge?
5. What observable behavior forces a separate path above that layer?

If question 5 is weak, the special case is probably at the wrong level.

## Observable Divergence

Separate paths are justified by differences such as:

- type rules
- evaluation order
- error behavior
- lowering shape
- metadata requirements
- runtime dispatch or execution behavior
- optimization legality

Different syntax or AST node spelling is not enough by itself.

## Forbidden Patterns

- Special-casing a construct only because it has a distinct AST node or surface
  spelling.
- Adding a parallel lowering or runtime path without naming the semantic
  divergence that requires it.
- Reusing a local implementation detail to justify a broader bad pattern.
- Collapsing truly distinct constructs into one path past the point where their
  observable semantics diverge.

## Validation Expectations

- Add paired tests for related constructs when the bug involves an unnecessary
  split or shared-family behavior.
- If two constructs are meant to share behavior, test both through the shared
  path.
- If they diverge, add tests that prove the divergence occurs exactly where the
  design says it should.
