# Nova Draft Critique Checklist

Use this checklist for architecture drafts and implementation plans before human
review.

Read `../../references/nova-delivery-checks.md` first when the draft is for a
Nova multi-session project.

## Architecture Drafts

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
