# Document Quality Standards

These standards apply to architecture documents produced or reviewed by
agents. They exist because agents default to terse bullet lists and keyword
phrases unless told otherwise, and those defaults produce documents that
are technically present but not useful for guiding implementation.

## Summary vs Detail

Every architecture document has two audiences reading at different speeds:

1. **Orientation readers** need the executive summary and process view to
   understand what the component does, what its contract is, and what the
   key invariant is. This takes under two minutes.

2. **Implementation readers** need the component architecture, phase
   details, and common change patterns to know where to put code, what
   invariants to maintain, and what else must change. This is a careful
   read of the relevant sections.

The document must serve both. The executive summary is not a condensed
version of the detail — it is a different kind of information. It answers
"what is this and why does it matter" while the detail answers "how does
it work and how do I change it."

## Prose Requirements

### Reasoning, not just facts

Every architectural decision should be accompanied by its rationale. State
what the rule is, then explain why it exists and what goes wrong when it
is violated.

Bad: "TypeMap is authoritative for expression types."

Good: "TypeMap is authoritative for expression types. Later phases must not
recover expression types from AST shape or RHS values, because those
secondary sources can diverge from the resolved type when inference,
coercion, or substitution have intervened. Consumer-side recovery from
secondary evidence is the single most common source of semantic bugs in
the frontend."

The reasoning is what lets an agent handle edge cases correctly. Without
it, the agent follows the rule literally in obvious cases and breaks it
in subtle ones.

### Paragraphs for architecture, lists for enumerations

Use paragraphs when explaining how a component works, why a boundary
exists, or what an invariant prevents. Use bullet lists when enumerating
concrete items (types in a catalog, fields in a record, validation
checks). Do not use bullet lists for architectural reasoning — the
compression loses the causal connections that make the reasoning useful.

### Callout pattern for scannable invariants

Use bold-prefixed callouts for information that agents need to find
quickly by scanning:

- **Invariant:** — what must always be true
- **Not responsible for:** — what this component does not do
- **Input:** / **Output:** — phase contracts

These callouts are scannable landmarks. The prose around them provides
the reasoning. Both are needed.

## Required Sections and Their Purpose

| Section | Purpose | Quality test |
|---|---|---|
| Executive Summary | Orient the reader in under 2 minutes | Can an agent who reads only this section correctly identify where a bug belongs? |
| Process View | Show the pipeline or workflow at a glance | Can an agent trace the flow of data from input to output? |
| Component Architecture | Define ownership, boundaries, invariants | Does every component have owned state, invariant, and not-responsible-for? |
| Supporting Abstractions | Describe internal data structures | Same ownership/invariant treatment as components? |
| Common Change Patterns | Prevent incomplete implementations | For each common change type, does the agent know every file to touch? |
| Error/Missing-Data | Prevent fallback shortcuts | Are forbidden patterns named with concrete examples? |

## Anti-Patterns

### The keyword document

A document where every section is a bulleted list of terms or short
phrases. This passes a presence check ("does the document cover X?") but
fails a usefulness check ("can an agent implement correctly using this?").

### The specification-as-architecture document

A document that exhaustively lists every variant, field, and enum case
instead of describing the architectural role and pointing to a spec
document for the full schema. Architecture describes boundaries and
invariants. Specification describes exhaustive shapes.

### The missing-change-patterns document

A document that describes the system thoroughly but does not tell the
reader how to modify it. The agent reads it, understands the architecture,
then makes an incomplete change because it didn't know which other
components also needed updating.

### The prose-free invariant

An invariant stated without reasoning: "Invariant: all nodes have types."
This tells the agent what to check but not why it matters or what to do
when it's violated. Add the reasoning: what breaks downstream, what the
failure mode looks like, what the forbidden recovery path is.
