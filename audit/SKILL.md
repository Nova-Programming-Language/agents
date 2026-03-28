---
name: audit
description: Audit code changes for correctness, structure, and forward impact; use after implementation to verify acceptance criteria, check architecture and abstraction, and flag risks.
---

# Audit

## When to Use

After any implementation. Works standalone or in the orchestrate pipeline.

## Dimensions

Every audit evaluates three dimensions. All must be clean to pass.
Criteria details are in `references/structural-checks.md`.

1. **Functional** — does the code meet acceptance criteria?
2. **Structural** — architecture, abstraction, completeness, integrity
3. **Forward impact** — does this help or hinder upcoming work?

## Inputs

1. **What was asked** — from BRIEF.md, commit message, or user description
2. **What changed** — `git diff [base]..HEAD`
3. **What comes next** — from PLAN.md or user context

## Output

Append a dated section to `project-notes/<slug>/AUDIT.md` per
`references/audit-template.md`. AUDIT.md is append-only.

The audit agent reports findings. It does not fix code.

## References

- `references/audit-prompt.md` — agent prompt template
- `references/audit-template.md` — output format
- `references/structural-checks.md` — all evaluation criteria
