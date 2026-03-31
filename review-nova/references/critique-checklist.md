# Nova Draft Critique Checklist

Use this checklist for architecture drafts and implementation plans before human
review.

Read `../../references/nova-delivery-checks.md` first when the draft is for a
Nova multi-session project.

Read `../../design/references/document-quality.md` for prose and structure
standards.

## Architecture Drafts

### Document structure

- Does the document have an executive summary? Not a one-line purpose
  statement, but 2-4 paragraphs that answer: what does this component do,
  what is its output contract, what is the key invariant, and what is the
  most common implementation mistake?
- Does the document have a process view or pipeline diagram when the
  component has sequential stages?
- Does every component section have **Invariant:** and **Not responsible
  for:** callouts?
- Does the document have a Common Change Patterns section? This is not
  optional — it is the single highest-leverage section for preventing
  incomplete implementations.
- Does the document have an explicit Missing-Data Behavior or Forbidden
  Fallbacks section?

### Prose quality

- Are architectural decisions accompanied by reasoning (why it exists, what
  goes wrong when violated), or are they stated as bare facts?
- Are component descriptions written in prose paragraphs, or are they just
  bulleted keyword lists? Bullet lists are for enumerations (types, fields,
  checks), not for architectural reasoning.
- Are invariants explained with enough context that an agent can handle edge
  cases, or are they bare assertions ("all nodes have types")?
- Does the document explain *why* boundaries are where they are, not just
  *what* they are?

### Abstraction design

- For each abstraction boundary: is the owned state named, the invariant
  stated, the hidden details identified, the negative boundary drawn, and
  the failure mode specified?
- Are component boundaries justified by what changes independently, or are
  they arbitrary groupings?
- Where a component says "not responsible for," is that boundary clear
  enough that an agent would not accidentally put the wrong logic there?

### Technical completeness

Check for:

- missing or weak contract definition
- missing affected Nova command or execution surfaces
- missing authoritative spec or architecture doc links
- unclear source-of-truth ownership
- missing required-data behavior
- missing loud-failure behavior for invalid or missing required state
- missing metadata contract, `prelude.novi`, runtime ABI, or compiled-backend
  implications when relevant
- fallback or heuristic-friendly wording
- syntax-shaped special casing where a shared semantic family should exist
- unclear divergence points between related constructs
- missing validation or invariants
- missing cross-surface parity expectations
- phase sequencing that is ambiguous or unrealistic

## Implementation Plans

Check for:

- milestones that are too vague or too large
- sequencing that ignores architecture dependencies
- missing cross-command impact
- missing affected Nova command or execution surfaces
- missing authoritative doc links
- proof defined as test directories instead of exact commands and checks
- missing negative validation
- missing loud-failure validation for invalid or missing required state
- missing interpreter / compiled parity checks when relevant
- missing build or runtime prerequisite calls when relevant
- lack of blocker handling
- lack of clear next-step granularity for the next coding session
- milestones whose implementation approach implies multi-concern functions
  instead of decomposed steps

## Verdict

Use one of:

- `ready_for_human_review`
- `revise_before_human_review`

## Recording Guidance

Record:

- major findings
- revisions applied or still needed
- verdict

If the draft is already strong, say so plainly and note any residual risks
instead of inventing low-value issues.
