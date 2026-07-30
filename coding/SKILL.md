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
2. `../references/package-onboarding.md` when modifying a package repository
   or package directory
3. the relevant spec files under `docs/specs/`
4. the relevant architecture docs under `docs/architecture/`
5. `docs/architecture/semantic-families.md` when a change may affect a named family across frontend, lowering, and backends
6. `../references/source-of-truth.md`
7. `../references/artifact-freshness.md`
8. `../references/runtime-evidence.md` for reproducible runtime bugs
9. `../references/semantic-family.md` when related constructs may share a family
10. `references/nova-semantics.md` when language semantics matter
11. the affected code paths

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
- Report state truthfully: `scaffolded`, `implemented-but-not-exposed`,
  `exposed`, or `release-verified`. Never call a partial state complete.
- Do not expose a public API or configuration variant until all behavior
  promised at that release boundary is implemented and tested across required
  engines/surfaces, unless the approved plan defines a real independently
  usable release boundary.
- No production placeholders: `pass`, TODO/unimplemented markers, placeholder
  panics, unconditional unsupported results, empty-success responses, ignored
  errors, or test-only dispatch for committed contract paths. Search for them,
  then semantically inspect every branch; grep alone cannot prove completeness.
- For package changes, complete `../references/package-onboarding.md` before
  editing source, interfaces, manifests, generated docs, examples, or tests.
- No fallbacks, heuristics, or silent degradation to hide missing data.
- Do not work around bugs in another component — fix the producer.
- Stay within the component boundaries from the architecture document.
- Rebuild artifacts before validation.
- Gather runtime evidence before changing code for reproducible bugs.
- Check for a shared semantic family before adding construct-specific paths.
- If `docs/architecture/semantic-families.md` names the affected family, update
  the whole family up to the first explicit divergence point; do not
  special-case one member above that layer.
- Do not route a few built-in or familiar types through a direct path while all
  remaining types use a generic fallback unless the semantic divergence is
  explicit and documented at that layer.
- Keep functions to a single responsibility. Decompose beyond ~50 lines
  or 3+ nesting levels into named steps at genuine responsibility boundaries.
- Refactoring should result in a net reduction in lines of code. If a
  refactor adds more code than it removes, it is adding complexity, not
  reducing it.
- Do not commit formatting-only changes. Reformatting lines you didn't
  otherwise modify creates diff churn that obscures real changes and
  pollutes git blame.
- Code and test changes must not introduce warnings on the affected build or
  crate-test surfaces. Treat new warnings as failures and either fix them or
  report an explicit blocker.

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

- Use targeted tests repeatedly while implementing.
- Run the owning component's suite once at the milestone gate.
- Run broader/canonical regression only at approved production-changing
  milestone gates and final release, not after each file or checkpoint.
- Use `scripts/run-nova-full-regression.sh check` for full-suite regression.
- Use `cargo check -p <crate>` / `cargo test -p <crate>` for crate-local.
- Prefer a warning-deny check such as `RUSTFLAGS="-D warnings" cargo test
  -p <crate> --no-run` when the change touches Rust code or tests that emit
  warnings only under test compilation.
- Use `../references/artifact-freshness.md` when consuming release artifacts.

## Architecture-Driven Work

For evolution phases: read the phase plan, implement the whole requirement,
do not reintroduce deprecated fallbacks, record blockers explicitly.

## Depth-First Work

Complete each fix fully (diagnose, fix, verify) before starting the next.
Keep sequential failures within the current milestone unless evidence proves
independent root causes. Do not invent subphases because of time, file count,
context size, or a failing test.

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
