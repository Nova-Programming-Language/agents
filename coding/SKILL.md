---
name: coding
description: Implement or modify the Nova toolchain safely; use when changing interpreter, frontend, codegen, CLI, tests, or architecture-driven behavior in the Nova repo.
---

# Coding

## When to Use

Use this skill for Nova repo implementation work — respecting semantic
constraints, architecture documents, and cross-command behavior.

## Source of Truth

Read in this order:

1. `docs/agent-rules.md`
2. the relevant spec files under `docs/specs/`
3. the relevant architecture docs under `docs/architecture/`
4. `../references/source-of-truth.md`
5. `../references/artifact-freshness.md`
6. `../references/runtime-evidence.md` for reproducible runtime bugs
7. `../references/semantic-family.md` when related constructs may share a family
8. `references/nova-semantics.md` when language semantics matter
9. the affected code paths

If the spec or architecture is ambiguous, stop and ask.

## Using Architecture Documents

Before implementing, read the relevant architecture document to answer:

1. **Which component owns this?** Check owned state, invariants, and
   "Not responsible for" boundaries.
2. **What else must change?** Follow the Common Change Patterns section
   for your type of modification.
3. **What is forbidden?** Check Missing-Data Behavior / Forbidden Fallbacks.
   Reconstructing data from secondary evidence is a forbidden shortcut.

Flag missing Common Change Patterns coverage as a documentation gap.

## Implementation Rules

- Implement the spec and architecture, not just the test expectation.
- No fallbacks, heuristics, or silent degradation to hide missing data.
- Do not work around bugs in another component — fix the producer.
- Stay within the component boundaries from the architecture document.
- Rebuild artifacts before validation.
- Gather runtime evidence before changing code for reproducible bugs.
- Check for a shared semantic family before adding construct-specific paths.
- Keep functions to a single responsibility. Decompose beyond ~50 lines
  or 3+ nesting levels into named steps at genuine responsibility boundaries.
- Refactoring should result in a net reduction in lines of code. If a
  refactor adds more code than it removes, it is adding complexity, not
  reducing it.
- Do not commit formatting-only changes. Reformatting lines you didn't
  otherwise modify creates diff churn that obscures real changes and
  pollutes git blame.

## Naming

Names are the primary documentation.

- **Functions**: verb phrase. `resolve_type` not `do_type`. Two things
  needing "and" means two functions.
- **Variables**: noun phrase. `remaining_attempts` not `n`. Loop counters fine.
- **Booleans**: predicate form. `is_resolved`, `has_errors`. Not `flag`.
- **Modules/files**: noun describing ownership. `type_resolver` not `utils`.
- **Consistency**: match existing patterns. Don't introduce `fetch_X` if
  the codebase uses `resolve_X`.

Avoid: `data`, `result`, `tmp`, `val`, `info`, `manager`, `handle`,
`process`. No abbreviations that lose meaning. No misleading names.

## Comments

Comments explain *why*, not *what*.

- Comment: non-obvious intent, business rules, workarounds, magic numbers,
  safety invariants the type system doesn't enforce.
- Don't comment: code restatements, obvious types, clear control flow.
- Keep comments near the code. Update or remove when changing nearby code.
- No TODO/FIXME in new code — do it or file an issue.

## Source-of-Truth Contract

Before implementing a lookup or decision path, state the answer being
computed, its authoritative source, whether missing data is valid or a bug,
and which component owns producing it.

Forbidden: string/pattern heuristics when structured data exists, fallback
from contract data to AST/registries, defaulting required data to None/empty,
"temporary" compatibility logic.

## Semantic-Family Check

Before adding a construct-specific path, state the semantic family, the
closest existing construct, whether new is identical/specialization/sibling,
and the first layer where semantics diverge. Prefer shared core with thin
specialization over separate paths.

## Component Workflow

1. State the semantic rule being changed.
2. Identify the owning component.
3. Identify every user-facing path affected.
4. For runtime bugs, collect runtime evidence before broad code reading.
5. For related constructs, find the highest shared semantic layer first.
6. Implement the general fix.
7. Validate all impacted paths, not just one.

## Cross-Command Audit

For shared behavior, audit: `nova run`, `nova check`, `nova repl`,
`nova test`, `nova doc`, `nova-test-runner`, compiled backend paths.

## Validation Strategy

Before validation: name the command, identify which artifacts it executes,
rebuild if needed, state what was rebuilt.

- Smallest command that proves the change.
- Then the owning component's test suite broadly.
- Then broader commands if the feature crosses subsystems.
- Use `scripts/full-regression.sh check` for full-suite regression.
- Use `cargo check -p <crate>` / `cargo test -p <crate>` for crate-local.
- Use `../references/artifact-freshness.md` when consuming release artifacts.

## Architecture-Driven Work

For evolution phases: read the phase plan, implement the whole requirement,
do not reintroduce deprecated fallbacks, record blockers explicitly.

## Depth-First Work

Complete each fix fully (diagnose, fix, verify) before starting the next.
No batching. If 3+ independent items, use `fix-test` for sequencing.

## When Invoked by Orchestrate

Brief and plan take precedence. Read BRIEF.md, follow its approach, update
STATUS.md when done or blocked. If the brief conflicts with source-of-truth
rules, record the conflict and follow the brief — the audit catches problems.

## References

- `../references/source-of-truth.md`
- `../references/artifact-freshness.md`
- `../references/runtime-evidence.md`
- `../references/semantic-family.md`
- `references/nova-semantics.md`
