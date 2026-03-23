---
name: review-nova
description: Critique Nova architecture drafts, implementation plans, and major project artifacts before human review; use after drafting and before requesting or acting on human feedback.
---

# Review Nova

## When to Use

Use this skill after drafting a Nova architecture document or implementation
plan and before requesting or acting on human feedback.

Typical targets:

- architecture docs in `docs/architecture/`
- `project-notes/<slug>/PLAN.md`
- other major project artifacts that define contracts, sequencing, or proof

## Read First

- `docs/agent-rules.md`
- `../references/source-of-truth.md`
- `../references/semantic-family.md`
- `../references/nova-delivery-checks.md`
- `references/critique-checklist.md`

## Non-Negotiable Rules

- The critique pass happens after drafting and before human feedback is
  requested or acted on.
- The critique should surface findings, risks, ambiguities, and missing proof,
  not rewrite the draft into implementation detail.
- Findings should be specific enough that the draft can be revised before human
  review.
- For Nova work, the critique must check command surfaces, contract ownership,
  loud-failure behavior, and proof commands explicitly.
- If the draft is already strong, record that explicitly instead of inventing
  weak findings.

## Workflow

1. Read the draft and its linked PRD, project backbone, and architecture docs
   as needed.
2. Critique it using `references/critique-checklist.md`.
3. Record one of:
   - ready for human review
   - revise before human review
4. Apply or record the resulting revisions before moving on.
5. Update the project artifacts so the critique gate is visible across sessions.

## Output Shape

The critique should cover:

- findings ordered by severity
- open questions or ambiguities
- validation or proof gaps
- readiness verdict for human review

## Open These References As Needed

- `../references/nova-delivery-checks.md`
- `references/critique-checklist.md`
