---
name: audit
description: Audit code changes for functional correctness, structural quality, and forward impact; use after any implementation to verify acceptance criteria, check architecture and abstraction, identify semantic duplication, and flag risks to upcoming work.
---

# Audit

## When to Use

Use after any implementation to verify the changes are correct, well-structured,
and won't create problems downstream. Works standalone or as part of the
orchestrate pipeline.

## Three Dimensions

Every audit evaluates three dimensions. All three must be clean for the
audit to pass.

### Functional

Does the code do what was asked?

- Each acceptance criterion gets an explicit pass/fail with evidence
- Validation uses exact commands and observable behavior, not inspection alone
- If a criterion cannot be evaluated, it is marked `cannot_verify` with reason

### Structural

Is the code well-placed and well-shaped?

- **Architecture**: Changes are in the correct layer and component. Ownership
  boundaries are respected.
- **Abstraction**: Semantically similar code shares an abstraction. No
  near-duplicates that should be unified. No premature abstractions either.
- **Semantic families**: Constructs that should share a path still do.
  New divergence points are flagged.
- **Patterns**: New code follows the conventions of the files it touches.
- **Scope**: No out-of-scope changes unless justified.

### Forward Impact

Does this make the next piece of work easier or harder?

- Interface shapes that downstream work will depend on
- Coupling that will constrain future changes
- Decisions baked in that may need undoing
- Small fixes now that prevent larger rework later

## Inputs

The audit agent needs:

1. **What was asked** — acceptance criteria and constraints. Sources:
   - `project-notes/<slug>/BRIEF.md` (if orchestrating)
   - A user-provided description of what the change should do
   - The commit message or PR description
2. **What changed** — the diff. Via `git diff` or `git diff [base]..HEAD`.
3. **What comes next** — upcoming work context. Sources:
   - `project-notes/<slug>/PLAN.md` (if orchestrating)
   - User description of planned follow-up work
   - If unknown, evaluate based on likely extension points

If `project-notes/<slug>/BRIEF.md` exists, use it. Otherwise, construct the
acceptance criteria from whatever context is available.

## Running the Audit

### Standalone

Read `references/audit-prompt.md` and construct the agent prompt with:
- The project slug (from `project-notes/`)
- The acceptance criteria (from brief, user, or commit message)
- The base commit for the diff
- Any upcoming work context

Spawn a general-purpose agent with the constructed prompt.
The agent appends a dated section to `project-notes/<slug>/AUDIT.md`.

### From Orchestrate

The orchestrate skill references this skill's prompt and templates
directly. See `orchestrate/references/agent-briefs.md`.

## Output

The audit agent appends a dated section to `project-notes/<slug>/AUDIT.md`
using the format in `references/audit-template.md`. AUDIT.md is
append-only — each audit adds a new dated section; prior sections remain
as history.

A milestone or change does not pass until all three dimensions are clean.
Structural issues are never deferred as tech debt — they are fixed or
escalated.

## Read As Needed

- `references/audit-prompt.md` — the full agent prompt template
- `references/audit-template.md` — AUDIT.md section format
- `references/structural-checks.md` — detailed structural criteria
