---
name: design
description: Write or update Nova architecture documents in docs/architecture; use when planning or refactoring a component before implementation.
---

# Design

## When to Use

Use this skill when creating or revising architecture documents in
`docs/architecture/` for a new feature, a refactor, or an implementation
evolution plan.

## Inputs

Read:

1. the relevant language spec files in `docs/specs/`
2. related architecture documents in `docs/architecture/`
3. `docs/agent-rules.md`
4. `../references/source-of-truth.md`
5. `../references/semantic-family.md`

If another architecture document consumes the same public interface, audit it
for compatibility before finalizing the new design.

## What an Architecture Document Is For

An architecture document is the primary tool for preventing implementation
drift, consumer-side recovery, and incomplete changes. It exists so that a
coding agent reading it can answer three questions without looking at the
code:

1. **Where does this work belong?** Which component owns the state, the
   invariant, and the transition being implemented?
2. **What must not happen?** Which shortcuts, fallbacks, and consumer-side
   recoveries are forbidden, and why?
3. **What else must change?** If I modify this component, which other
   components must be updated in the same change?

A document that does not answer these questions is not yet an architecture
document — it is a set of notes.

## Abstraction Design

Before writing the document, identify the abstractions. An abstraction is a
component boundary where one side (the producer) owns semantic facts and
the other side (the consumer) relies on those facts without re-deriving
them.

For each abstraction in the design:

1. **Name the boundary.** What is on each side? What crosses?
2. **Name the owned state.** What data does the producer create that the
   consumer reads? Be specific — name the types, tables, or registries.
3. **Name the invariant.** What must always be true about that state after
   the producer finishes? The invariant should be concrete enough that a
   validator could check it.
4. **Name the hidden details.** What does the producer know that the consumer
   must not depend on? Implementation details, intermediate representations,
   internal ordering — state what is *not* part of the contract.
5. **Name the negative boundary.** What is the producer explicitly *not*
   responsible for? This prevents agents from extending the producer's scope
   or putting consumer logic in the producer.
6. **Name the failure mode.** When the producer's required data is missing or
   invalid, what happens? The answer should be "fail loudly" or "force a
   producer fix," not "reconstruct from secondary evidence."

If you cannot answer these questions clearly, the abstraction boundary is
not well-defined. Redesign before writing.

## Document Structure

Architecture documents should follow this structure. Each section has a
specific purpose for agent consumption.

### 1. Header and Cross-References

Status, last-updated date, and links to related architecture docs, normative
schema references, and spec documents. An agent entering the document should
immediately know what adjacent documents to read and which document is
authoritative for what.

### 2. Executive Summary

A 2-4 paragraph overview that answers: what does this component do, what is
its single output contract, what is the most important invariant, and what is
the most common implementation mistake in this area.

This section exists so an agent can read it and have the right mental frame
for everything that follows. It is not a table of contents or a purpose
statement. It should explain the *why* behind the architecture — what
problem it solves and what failure modes it prevents.

Write this section last, after the detail sections are complete, so it
accurately summarizes what the document actually says.

### 3. Process View

If the component has a pipeline or sequential workflow, describe it as a
sequence of state transitions. A diagram is valuable here. After the
diagram, explain in prose what each stage produces and what the overall
pipeline guarantees.

The process view is the bird's-eye orientation. It should be readable in
under a minute.

### 4. Component Architecture

The core of the document. For each component or sub-component, write a
section that includes:

- **What it owns** — the specific states, tables, or registries it creates
  or maintains. Name the types.
- **What it reads** — the inputs from earlier components.
- **What it writes** — the outputs consumed by later components.
- **Invariant** — bold callout. What must always be true about its output.
  Concrete enough to validate.
- **Not responsible for** — bold callout. What this component explicitly does
  not do. This is as important as what it does — it prevents scope creep and
  misplaced logic.

Write each component section in prose, not just bullet points. Explain *why*
the invariant matters and *why* the boundary is where it is. An agent that
understands the reasoning can make correct judgment calls in edge cases; an
agent that only has a rule will break it when the situation is slightly
different.

### 5. Key Internal Abstractions

For supporting data structures and engines that do not cross the output
boundary but are essential to understanding the component's internals.
Each gets the same treatment: owned state, invariant, not-responsible-for.

### 6. Detailed Phase or Subsystem Sections

Expand the process view into per-phase or per-subsystem detail. Each section
should state its input, output, and the specific rules or behaviors that
apply. Keep these at the architecture level — describe what the phase does
and what rules it enforces, not the exhaustive enumeration of every variant
or field. Point to specification documents for exhaustive schema details.

### 7. Common Change Patterns

**This section is not optional.** It is the single highest-leverage section
for preventing incomplete implementations.

For each common type of modification (adding a new form, changing a rule,
modifying a contract, adding a declaration), describe which components must
be updated and in what order. An agent reading this section should know
exactly which files and phases to touch for a given kind of change without
having to reason about the full architecture from scratch.

### 8. Error Model and Missing-Data Behavior

How errors propagate, what happens when required data is missing, and which
fallback behaviors are explicitly forbidden. Be specific — name the
forbidden patterns and explain why they are forbidden with concrete examples
of what goes wrong.

### 9. Cross-Document Concerns

Import/export behavior, cross-module compilation, or any behavior that spans
multiple architecture documents. Reference the authoritative document for
each shared concern.

## Prose Quality

Architecture documents must use prose, not just keywords or bullet lists.
Every section should explain reasoning, not just state facts. The difference:

**Weak (keyword-style):**
> - owns TypeMap
> - invariant: resolved metadata available

**Strong (prose with reasoning):**
> The constraint system owns `TypeMap` and `Substitution`. It reads `Module`,
> `TypeEnv`, and `TraitRegistry`, and writes resolved node-level semantic
> facts. Later phases obtain resolved facts from `TypeMap` and do not recover
> them from syntax, RHS values, or backend symbol names — this is the
> invariant that prevents consumer-side recovery throughout the rest of the
> pipeline.

The strong version tells the agent *what* the invariant is, *why* it exists,
and *what it prevents*. The weak version states a fact that an agent may or
may not apply correctly in edge cases.

Bullet lists are appropriate for enumerations (list of types in a catalog,
list of validation checks), not for architectural reasoning.

## Design Rules

- Start from the contract the rest of the system should consume.
- Do not expose temporary implementation shortcuts as part of the public
  design.
- Keep interfaces authoritative and implementation details replaceable.
- For each contract decision, state which component is the source of truth
  and whether missing data is valid or a hard error.
- Do not justify separate paths with syntax alone; identify the highest
  shared semantic layer first.
- If two constructs are in the same semantic family, say whether they share
  one internal form, a shared core with specialization, or a true split.
- If they split, state the first layer where observable semantics diverge
  and what behavior forces that split.
- If a design would permit secondary inference, string heuristics, or silent
  degradation, say that it is forbidden unless the design intentionally
  makes the state optional.
- Before requesting or acting on human feedback for an architecture draft,
  run `review-nova` on the draft and address or record its findings.
- If a spec is missing a semantic rule, pause and ask instead of freezing
  the ambiguity into architecture.

## Cross-Document Audit

When the design touches an existing interface:

- inspect the producer document
- inspect the consumer documents
- identify mismatches explicitly
- tell the human where related docs may also need updates

Sub-agents are useful for document audits when there are multiple adjacent
architecture docs.

## Handoff Expectation

A finished architecture doc should make implementation sequencing obvious:

- what component changes first
- which invariants must hold
- which validation proves the design is implemented
- which related constructs must stay unified and which must stay distinct
- which common change patterns apply to the work about to be done

## Exemplar

`docs/architecture/frontend-architecture.md` demonstrates the expected
quality bar. It has an executive summary, component architecture with
invariant/not-responsible-for callouts, process view, detailed phase
sections, supporting internal abstractions, common change patterns, and
missing-data behavior. Use it as a reference when writing new architecture
documents.

## References

- `references/document-quality.md` — prose and structure standards
