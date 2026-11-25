---
name: audit
description: Audit finished work against its brief, plan and architecture docs, appending the verdict to AUDIT.md; use after implementation, before commit. Not for drafts or plans (use critique).
---

# Audit

## When to Use

After any implementation, before it is committed or the milestone is closed.
Works standalone, from `fix-test` step 6, or in the orchestrate pipeline.

## Dimensions

Every audit evaluates three dimensions. All must be clean to pass.
Criteria are in `references/structural-checks.md`.

1. **Functional** — does the code meet every acceptance criterion and the
   rest of the brief? (Completeness)
2. **Structural** — architecture, repairs, abstraction, semantic families,
   patterns, function size, integrity, refactoring, scope
3. **Forward impact** — does this help or hinder upcoming work?

A repair is clean only when it sits in the component whose contract was
violated — even when every test passes. See `../references/failure-attribution.md`.

Never certify a tested subset while original acceptance criteria remain.
Report one truthful state: `scaffolded`, `implemented-but-not-exposed`,
`exposed`, or `release-verified`. Only the last counts as fully complete.

### Pre-existing problems

A problem counts against this change unless it is pre-existing. It is
pre-existing only if the approved brief, design, or rule text already allowed
it *and* it was already present at the project's recorded base. Never compare
against the previous fix round's commit — that lets one round's defect pass
as pre-existing in the next.

## Inputs

1. **What was asked** — BRIEF.md, commit message, or user description
2. **What changed** — the diff from the base to the working tree; find the
   base and pick a focused or full depth per `references/scope-and-depth.md`.
   Depth never skips an audit: small changes are where shortcuts hide.
3. **What comes next** — PLAN.md or user context

## Output

Append a dated section to `project-notes/<slug>/AUDIT.md` (standalone:
`project-notes/fix-<issue-or-branch>/`) per `references/audit-template.md`.
The audit reports findings only: it never fixes code, and AUDIT.md is the
only file it writes. To run it as a separate agent, fill in
`references/audit-prompt.md`.
