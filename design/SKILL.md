---
name: design
description: Write or update the project's architecture documents (ownership, invariants, change patterns, missing-data behavior); use when planning or refactoring a component before implementation. Not for implementing code (use coding), PRDs (use prd), or critiquing a finished draft (use critique).
---

# Design

## When to Use

Creating or revising architecture documents in the project's architecture
docs directory (profile §Paths; find the profile per
`../references/project-profile.md`).

## Inputs

1. Relevant spec files (profile §Paths)
2. Related architecture documents (profile §Paths)
3. The project's rules file (profile §Rules)
4. `../references/source-of-truth.md`
5. `../references/semantic-family.md`

Audit adjacent architecture docs for compatibility before finalizing.

## Purpose of an Architecture Document

A coding agent reading the document should be able to answer, without looking
at code (why: `references/document-quality.md`, "Purpose"):

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

Write the nine sections in `references/document-quality.md`, "Document
Structure", in order:

1. Header and cross-references; add or update this doc's rows in the
   project's architecture map (`../references/architecture-docs.md`).
2. Executive summary (2-4 paragraphs, written last).
3. Process view.
4. Component architecture, with an **Invariant:** and a **Not responsible
   for:** callout per component.
5. Key internal abstractions, with the same callouts.
6. Detailed phase sections.
7. Common Change Patterns — never optional.
8. Error model and missing-data behavior, naming forbidden fallbacks with
   concrete examples.
9. Cross-document concerns.

Use prose for reasoning and bullet lists only for enumerations: state what,
why, and what it prevents (`references/document-quality.md`).

## Design Rules

- Start from the contract consumers should see.
- Don't expose implementation shortcuts as public design.
- For each decision, state the source of truth and missing-data behavior.
- Don't justify separate paths with syntax alone — find the shared
  semantic layer first.
- For semantic families, state whether they share one form, a core with
  specialization, or a true split at the first divergence layer.
- Forbid secondary inference and silent degradation explicitly.
- Implement each rule as one judgment called from every site it applies
  to, not as per-site copies.
- If a rule depends on a fact that the representation its consumers already
  read does not carry (a type, an IR node, a lookup key, a runtime value
  layout), the first change makes that representation carry it, or records
  the fact once, with a single producer, where every consumer reads it. A
  consumer that finds no record raises an internal error. Never re-derive the
  fact per site from other data, and never keep it in a parallel structure
  that each consumer must remember to consult.
- A decision made in one phase and needed in a later one (a value-changing
  conversion, a resolved target, an ownership choice) is recorded once where
  it is decided and applied at one point per consumer; no later phase or
  engine derives it again.
- Run `critique` before requesting human feedback.
- If a spec is ambiguous, ask — don't freeze ambiguity into architecture.

## Cross-Document Audit

When touching an existing interface: inspect producer and consumer docs,
identify mismatches, flag where related docs need updates.

## Handoff

A finished doc makes implementation sequencing obvious: what changes
first, which invariants hold, which validation proves completion, which
change patterns apply.

## Exemplar

The exemplar architecture doc named in profile §Paths demonstrates the
quality bar.

## References

- `references/document-quality.md`
