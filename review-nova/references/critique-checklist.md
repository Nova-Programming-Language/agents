# Nova Draft Critique Checklist

Use for architecture drafts and implementation plans before human review.
Read `../../references/nova-delivery-checks.md` first for multi-session
projects. Read `../../design/references/document-quality.md` for standards.

## Architecture Drafts

### Structure
- Executive summary (2-4 paragraphs, not a one-line purpose statement)?
- Process view or pipeline diagram?
- Every component has **Invariant** and **Not responsible for** callouts?
- Common Change Patterns section (not optional)?
- Missing-Data / Forbidden Fallbacks section?

### Prose quality
- Decisions have reasoning (why, what breaks when violated)?
- Component descriptions in prose paragraphs, not keyword bullets?
- Invariants explained with enough context for edge cases?
- Boundaries justified by what changes independently?

### Abstraction design
- Each boundary has: owned state, invariant, hidden details, negative
  boundary, failure mode?
- "Not responsible for" clear enough to prevent misplaced logic?

### Technical completeness
- Contract definition, affected command surfaces, authoritative doc links
- Source-of-truth ownership, required-data behavior, loud-failure behavior
- Metadata contract / runtime / compiled-backend implications
- No fallback-friendly wording, no syntax-shaped special casing
- Divergence points, validation/invariants, cross-surface parity
- Realistic phase sequencing

## Implementation Plans
- Milestones sized and sequenced correctly
- Cross-command impact, affected surfaces, authoritative links
- Proof as exact commands (not test directories)
- Negative validation, loud-failure validation
- Interpreter/compiled parity, build prerequisites
- Blocker handling, next-step granularity
- Steps don't imply multi-concern functions

## Verdict

`ready_for_human_review` or `revise_before_human_review`

Record major findings, revisions applied/needed, verdict. If the draft
is strong, say so — don't invent weak findings.
