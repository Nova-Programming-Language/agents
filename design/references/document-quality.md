# Document Quality Standards

Agents default to terse bullet lists. These standards exist to produce
documents that are actually useful for guiding implementation.

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
