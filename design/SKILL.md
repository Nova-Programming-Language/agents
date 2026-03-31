---
name: design
description: Write or update Nova architecture documents in docs/architecture; use when planning or refactoring a component before implementation.
---

# Design

## When to Use

Creating or revising architecture documents in `docs/architecture/`.

## Inputs

1. Relevant spec files in `docs/specs/`
2. Related architecture documents in `docs/architecture/`
3. `docs/agent-rules.md`
4. `../references/source-of-truth.md`
5. `../references/semantic-family.md`

Audit adjacent architecture docs for compatibility before finalizing.

## Purpose of an Architecture Document

An architecture document prevents implementation drift, consumer-side
recovery, and incomplete changes. A coding agent reading it should answer
without looking at code:

1. **Where does this work belong?** (component ownership)
2. **What must not happen?** (forbidden shortcuts)
3. **What else must change?** (co-required updates)

A document that cannot answer these is not yet architecture — it is notes.

## Abstraction Design

Before writing, identify the abstractions. For each boundary:

1. **Boundary**: what is on each side, what crosses
2. **Owned state**: specific types, tables, registries the producer creates
3. **Invariant**: what must always be true — concrete enough to validate
4. **Hidden details**: what consumers must not depend on
5. **Negative boundary**: what the producer is *not* responsible for
6. **Failure mode**: "fail loudly" or "fix the producer" — never
   "reconstruct from secondary evidence"

If these are unclear, the boundary is not well-defined. Redesign first.

## Document Structure

### 1. Header and Cross-References
Status, date, links to related docs and specs.

### 2. Executive Summary
2-4 paragraphs: what the component does, its output contract, key
invariant, most common implementation mistake. Write last. Not a
purpose statement — explain what the architecture prevents.

### 3. Process View
Pipeline diagram with prose explaining state transitions and guarantees.
Readable in under a minute.

### 4. Component Architecture
Core of the document. Per component:
- What it owns, reads, writes (name the types)
- **Invariant:** bold callout, concrete enough to validate
- **Not responsible for:** bold callout — prevents misplaced logic

Write in prose with reasoning, not just bullets. Explain *why* boundaries
are where they are.

### 5. Key Internal Abstractions
Supporting data structures that don't cross the output boundary. Same
ownership/invariant/not-responsible-for treatment.

### 6. Detailed Phase Sections
Per-phase input, output, rules. Architecture level — point to spec docs
for exhaustive schema details.

### 7. Common Change Patterns
**Not optional.** Highest-leverage section. For each common modification
type, list which components must be updated and in what order.

### 8. Error Model and Missing-Data Behavior
Error propagation, missing-data handling, forbidden fallback patterns
with concrete examples.

### 9. Cross-Document Concerns
Cross-module behavior referencing authoritative docs.

## Prose Quality

Use prose for reasoning, bullet lists for enumerations only.

**Weak:** `- owns TypeMap` / `- invariant: resolved metadata available`

**Strong:** "The constraint system owns `TypeMap`. Later phases obtain
resolved facts from it and do not recover them from syntax or symbol
names — this prevents consumer-side recovery throughout the pipeline."

State what, why, and what it prevents. See `references/document-quality.md`.

## Design Rules

- Start from the contract consumers should see.
- Don't expose implementation shortcuts as public design.
- For each decision, state the source of truth and missing-data behavior.
- Don't justify separate paths with syntax alone — find the shared
  semantic layer first.
- For semantic families, state whether they share one form, a core with
  specialization, or a true split at the first divergence layer.
- Forbid secondary inference and silent degradation explicitly.
- Run `review-nova` before requesting human feedback.
- If a spec is ambiguous, ask — don't freeze ambiguity into architecture.

## Cross-Document Audit

When touching an existing interface: inspect producer and consumer docs,
identify mismatches, flag where related docs need updates.

## Handoff

A finished doc makes implementation sequencing obvious: what changes
first, which invariants hold, which validation proves completion, which
change patterns apply.

## Exemplar

`docs/architecture/frontend-architecture.md` demonstrates the quality bar.

## References

- `references/document-quality.md`
