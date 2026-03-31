---
name: coding
description: Implement or modify the Nova toolchain safely; use when changing interpreter, frontend, codegen, CLI, tests, or architecture-driven behavior in the Nova repo.
---

# Coding

## When to Use

Use this skill for Nova repo implementation work, not for generic coding. It is
about respecting the repo's semantic constraints, architecture documents, and
cross-command behavior.

## Source of Truth

Read in this order:

1. `docs/agent-rules.md`
2. the relevant language spec files under `docs/specs/`
3. the relevant architecture docs and evolution plans under
   `docs/architecture/`
4. `../references/source-of-truth.md`
5. `../references/artifact-freshness.md`
6. `../references/runtime-evidence.md` for reproducible runtime bugs
7. `../references/semantic-family.md` when related constructs may share a
   semantic family
8. `references/nova-semantics.md` when language semantics matter
9. the affected code paths

If the spec or architecture is ambiguous, stop and ask instead of inventing a
 behavior.

## Using Architecture Documents

Before implementing, read the relevant architecture document in
`docs/architecture/`. Use it to answer:

1. **Which component owns this change?** Find the component whose owned
   state, invariant, or boundary covers the behavior you are modifying.
   If the architecture document has a "Not responsible for" section on
   that component, verify your change does not violate it.

2. **What else must change?** Check the "Common Change Patterns" section
   for the type of modification you are making. It will list which other
   components, tables, or validation steps must be updated alongside your
   primary change. Follow it — incomplete changes are the most common
   agent failure mode.

3. **What is forbidden?** Check the "Missing-Data Behavior" or "Forbidden
   Fallbacks" section. If your fix involves reconstructing data that
   should come from an upstream producer, that is a forbidden shortcut —
   fix the producer instead.

If the architecture document does not have a Common Change Patterns section
or does not cover your type of change, flag that as a documentation gap.

## Implementation Rules

- Implement the spec and architecture, not just the current test expectation.
- Do not add fallbacks, heuristics, silent degradation, or silent coercions to
  hide missing metadata or upstream bugs.
- Do not work around bugs in another component when the correct fix is to repair
  the producer.
- Keep changes within the component boundaries defined in the architecture
  document. If the architecture document names a component as "not responsible
  for" the behavior you are adding, your code belongs somewhere else.
- Rebuild the artifacts required by the validation command after source changes.
- For reproducible runtime bugs, gather runtime evidence before changing code.
- Do not special-case syntax before checking for a shared semantic family.
- Keep functions focused on a single responsibility. If a function you are
  writing or extending grows beyond ~50 lines or handles multiple distinct
  concerns (e.g., validation then transformation then persistence), decompose
  it into named steps. Each step should represent a genuine responsibility
  boundary, not an artificial split. Deep nesting (3+ levels) is a signal
  that the function is doing too much.

## Naming

Names are the primary documentation. A reader should understand what a
function does, what a variable holds, or what a module owns from the name
alone.

- **Functions**: verb phrase describing the action and result. `resolve_type`
  not `do_type`, `emit_diagnostic` not `handle`. If a function does two
  things that need "and" in the name (`validate_and_transform`), it should
  be two functions.
- **Variables**: noun phrase describing the content. `remaining_attempts` not
  `n`, `source_path` not `p`. Loop counters (`i`, `j`) and short-lived
  bindings in closures are fine.
- **Booleans**: predicate form. `is_resolved`, `has_errors`, `should_emit`.
  Not `flag`, `status`, `check`.
- **Packages/modules**: noun describing what the module owns. `type_resolver`
  not `utils`, `diagnostics` not `helpers`.
- **Files**: match the primary type or module they define. One concept per
  file when possible.
- **Consistency**: match the naming patterns already in the file and its
  neighbors. If the codebase uses `resolve_X` for lookups, do not introduce
  `fetch_X` or `get_X` for the same operation.

Avoid: generic names (`data`, `result`, `tmp`, `val`, `info`, `manager`,
`process`, `handle`), abbreviations that lose meaning (`ctx` used once,
`mgr`, `impl` as a variable name), and misleading names (a function named
`validate` that also transforms its input).

## Comments

Comments explain *why*, not *what*. The code already says what it does.

- **When to comment**: non-obvious intent, business rules that aren't
  self-evident from the code, workarounds with a reason, magic numbers
  with their derivation, safety invariants that the type system doesn't
  enforce.
- **When not to comment**: restating the code (`// increment counter`),
  explaining obvious types or signatures, narrating control flow that is
  clear from reading.
- **Keep comments near the code they explain.** A block comment 20 lines
  above the relevant code will become stale.
- **Update or remove comments when changing the code they describe.**
  A stale comment is worse than no comment — it actively misleads.
- **Do not add TODO/FIXME in new code.** If something needs doing, either
  do it or file an issue. TODOs in committed code are forgotten promises.

## Source-of-Truth Contract

Before implementing a lookup or decision path, state:

1. the exact answer being computed
2. the authoritative source for that answer
3. whether missing data is valid or a bug
4. which component owns producing or installing the data if it is missing

If the authoritative source should exist and does not, fail loudly or report
the blocker. Do not replace missing source-of-truth data with heuristics or
secondary inference.

Forbidden shortcuts:

- string-name or pattern heuristics when structured data exists or should exist
- fallback from contract data to AST, registries, or side maps because the
  primary source is missing
- defaulting required data to `None`, empty maps, or no-op behavior
- “temporary” compatibility logic that makes invalid states look valid

## Semantic-Family Check

Before adding a construct-specific path, state:

1. the semantic family the construct belongs to
2. the closest existing construct already covering that family
3. whether the new construct is identical, a specialization, or a sibling
4. the first layer where observable semantics actually diverge

If the construct is only a specialization of an existing family, prefer a thin
specialization over a shared core path. If it is truly distinct, split only at
the first layer where semantics diverge.

## Component Workflow

1. State the semantic rule you are changing.
2. Identify the owning component.
3. Identify every user-facing path the change affects.
4. If the bug is reproducible at runtime, do only enough code reading to find
   the likely debug surface, then collect runtime evidence.
5. If related constructs are involved, identify the highest shared semantic
   layer before adding any special case.
6. Implement the general fix.
7. Validate the impacted paths, not just one happy path.

For data-contract fixes, include the missing-data path in the implementation
plan, not just the success path.

## Cross-Command Audit Requirement

For shared toolchain/runtime behavior, audit:

- `nova run`
- `nova check`
- `nova repl`
- `nova test`
- `nova doc` example verification
- `nova-test-runner`
- compiled backend paths when applicable

Do not treat one command path as sufficient if the feature is shared.

## Validation Strategy

Artifact freshness checklist before validation:

1. Name the exact command you are about to run.
2. Identify whether it executes Cargo-built debug artifacts,
   `target/release/nova`, compiled/runtime-linked libraries, or x86 container
   artifacts.
3. Rebuild the required surfaces first, or explicitly note that the command
   itself guarantees freshness.
4. In your user-facing validation summary, state what was rebuilt or why no
   rebuild was needed.
5. In your user-facing implementation summary, state the authoritative source
   used and what happens if it is missing.

- Use the smallest command that proves the change.
- Then use a broader command if the feature crosses subsystems.
- For reproducible runtime bugs, stop broad code reading once the next useful
  fact is a live value or branch outcome that LLDB or narrow tracing can show.
- When removing a syntax-shaped special case, add paired tests for the related
  constructs that should share a path.
- When constructs truly diverge, add tests that prove the divergence occurs at
  the intended layer.
- When removing a fallback or heuristic, add a regression that proves missing
  required data fails loudly instead of silently degrading.
- Use `scripts/full-regression.sh check` for canonical full-suite regression.
- Use `cargo check -p <crate>` and `cargo test -p <crate> <filter>` for
  crate-local Rust validation.
- Use `../references/artifact-freshness.md` when the validation command may
  consume release or runtime artifacts.

## Architecture-Driven Work

If the change is part of an evolution phase:

- read the phase plan
- implement the whole phase requirement in the owned component
- do not reintroduce deprecated fallback paths
- record blockers explicitly when another component is not ready

## Depth-First Work

When presented with multiple problems, fixes, or changes:

- Complete each one fully before starting the next. Diagnose, fix,
  verify — then move on.
- A partially-fixed item is worse than an unfixed item. It creates the
  illusion of progress while hiding remaining work.
- "These are similar so I'll batch them" is not acceptable. Fix one,
  verify, then check if the others are resolved.
- If a fix list has more than 3 independent items, use the `fix-test`
  skill to enforce one-at-a-time sequencing.

## When Invoked by Orchestrate

When spawned as an implementation agent by `orchestrate`, the brief and
plan take precedence over independent architectural decisions:

- Read `project-notes/<slug>/BRIEF.md` for your task spec.
- Follow the architectural approach specified in the brief.
- Update `project-notes/<slug>/STATUS.md` when done or blocked.
- If the brief conflicts with source-of-truth rules, record the
  conflict in STATUS.md and follow the brief — the audit will catch
  genuine architectural problems.

## Open These References As Needed

- `../references/source-of-truth.md`
- `../references/artifact-freshness.md`
- `../references/runtime-evidence.md`
- `../references/semantic-family.md`
- `references/nova-semantics.md`
