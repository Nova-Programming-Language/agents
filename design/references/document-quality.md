# Document Quality Standards

Agents default to terse bullet lists. These standards exist to produce
documents that are actually useful for guiding implementation.

## Purpose

An architecture document prevents implementation drift, consumer-side
recovery, and incomplete changes; that is why it must answer the three
questions in `../SKILL.md` without the reader looking at code.

## Summary vs Detail

Two audiences: orientation readers (executive summary + process view,
under 2 minutes) and implementation readers (component architecture +
change patterns, careful read). The summary answers "what and why"; the
detail answers "how and what else."

## Prose Requirements

**Reasoning, not just facts.** State the rule, explain why, state what
breaks when violated.

Bad: "TypeMap is authoritative for expression types."
Good: "TypeMap is authoritative for expression types. Later phases must
not recover them from AST shape or RHS values, because those sources
can diverge after inference/coercion/substitution."

**Paragraphs for reasoning, lists for enumerations.** Don't use bullet
lists for architectural decisions — the compression loses causal
connections.

**Scannable callouts.** Use bold-prefixed **Invariant:**, **Not
responsible for:**, **Input:**/**Output:** as landmarks. Prose around
them provides reasoning. Both are needed.

## Document Structure

### 1. Header and Cross-References
Status, date, links to related docs and specs. Add or update this doc's rows
in the project's architecture map (the source paths it governs) — skills use
the map to find the doc for a changed file. See
`../../references/architecture-docs.md`.

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

## Prose Example

**Weak:** `- owns TypeMap` / `- invariant: resolved metadata available`

**Strong:** "The constraint system owns `TypeMap`. Later phases obtain
resolved facts from it and do not recover them from syntax or symbol
names — this prevents consumer-side recovery throughout the pipeline."

State what, why, and what it prevents.

## Required Sections

| Section | Quality test |
|---|---|
| Executive Summary | Can an agent identify where a bug belongs from this alone? |
| Process View | Can an agent trace data flow from input to output? |
| Component Architecture | Every component has owned state, invariant, not-responsible-for? |
| Common Change Patterns | Agent knows every file to touch for each change type? |
| Error/Missing-Data | Forbidden patterns named with concrete examples? |

## Anti-Patterns

- **Keyword document**: bullet lists of terms that pass presence checks
  but don't guide implementation.
- **Specification-as-architecture**: exhaustive variant/field listings
  instead of boundaries and invariants. Point to spec docs.
- **Missing change patterns**: thorough description, no modification guide.
- **Prose-free invariant**: "all nodes have types" without explaining
  what breaks downstream or what the forbidden recovery is.
