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

## Document Goals

A Nova architecture document should:

- define the public contract first
- hide implementation details behind stable abstractions
- name the owning components
- identify the authoritative source for each important answer or decision
- identify which constructs intentionally share a semantic family
- state where related constructs diverge and why
- state what happens when required data is missing
- call out disallowed fallbacks and heuristics explicitly
- call out non-goals explicitly
- describe migration and validation, not just the target state

## Recommended Structure

For most Nova architecture docs, include:

1. Purpose
2. Inputs / Outputs or public contract
3. Non-goals
4. Component boundaries
5. Runtime/data model if relevant
6. Execution or control-flow model if relevant
7. Migration or evolution phases
8. Validation / invariants
9. Missing-data behavior, shared semantic families, and forbidden fallbacks
10. Risks or open questions

## Design Rules

- Start from the contract the rest of the system should consume.
- Do not expose temporary implementation shortcuts as part of the public design.
- Keep interfaces authoritative and implementation details replaceable.
- For each contract decision, state which component is the source of truth and
  whether missing data is valid or a hard error.
- Do not justify separate paths with syntax alone; identify the highest shared
  semantic layer first.
- If two constructs are in the same semantic family, say whether they share one
  internal form, a shared core with specialization, or a true split.
- If they split, state the first layer where observable semantics diverge and
  what behavior forces that split.
- If a design would permit secondary inference, string heuristics, or silent
  degradation, say that it is forbidden unless the design intentionally makes
  the state optional.
- Before requesting or acting on human feedback for an architecture draft, run
  `review-nova` on the draft and address or record its findings.
- If a spec is missing a semantic rule, pause and ask instead of freezing the
  ambiguity into architecture.

## Cross-Document Audit

When the design touches an existing interface:

- inspect the producer document
- inspect the consumer documents
- identify mismatches explicitly
- tell the human where related docs may also need updates

Sub-agents are useful for document audits when there are multiple adjacent
architecture docs.

## Handoff Expectation

A finished Nova architecture doc should make implementation sequencing obvious:

- what component changes first
- which invariants must hold
- which validation proves the design is implemented
- which related constructs must stay unified and which must stay distinct
