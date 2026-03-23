---
name: delivery
description: Coordinate multi-session Nova projects from requirements through validation; use when work spans multiple commits or coding sessions and needs persistent project artifacts, stage gates, and required links to PRD, architecture, and implementation plan.
---

# Delivery

## When to Use

Use this skill when a Nova task spans multiple commits or sessions and needs
persistent project context, explicit lifecycle stages, and stable artifacts that
keep the agent focused on the right level of detail.

## Core Artifacts

Use a project folder:

```text
project-notes/<slug>/
  PROJECT.md
  PLAN.md
  STATUS.md
```

Artifact roles:

- `PROJECT.md`: durable project backbone
- `PLAN.md`: implementation plan and milestone breakdown
- `STATUS.md`: current tactical checkpoint for the next session

`PROJECT.md` must contain required links to:

- PRD
- architecture docs
- implementation plan

If one link does not apply, write `N/A` with a reason instead of leaving it
implicit.

## Read First

- `docs/agent-rules.md`
- `../references/source-of-truth.md`
- `../references/runtime-evidence.md`
- `../references/semantic-family.md`
- `../references/nova-delivery-checks.md`
- `references/stage-checklists.md`
- `references/project-template.md`
- `references/plan-template.md`
- `references/status-template.md`

## Session Start Rule

At the start of a session:

1. Read `PROJECT.md`.
2. Read `PLAN.md`.
3. Read `STATUS.md`.
4. Confirm the current lifecycle stage before coding.

Do not start by rereading old chats, commits, or broad code surfaces unless the
project artifacts are stale or incomplete.

## Required Stage Routing

- `Requirements`: `delivery`, plus `prd` when the project needs a real PRD
- `Architecture`: `delivery`, plus `design` when semantics, contracts, or
  interfaces change, then `review-nova` before human feedback
- `Implementation Plan`: `delivery`, then `review-nova` before human feedback
- `Implementation`: `coding` required; `building` and `docs` as needed
- `Validation`: `regression-test` and `building` required; `debug-nova` when a
  reproducible runtime failure needs live evidence
- Commit/push after a validated milestone: `checkin`

## Workflow

1. Create or refresh `PROJECT.md`, `PLAN.md`, and `STATUS.md`.
2. Identify the current stage and load the matching checklist from
   `references/stage-checklists.md`.
3. Use the required skill(s) for that stage.
4. Update the project artifacts before ending the session or changing stage.
5. Do not move to the next stage until the current stage exit criteria are met
   or an explicit blocker is recorded.

## Non-Negotiable Rules

- `PROJECT.md` is the durable source of project truth.
- `PLAN.md` is the source of truth for milestone sequencing.
- `STATUS.md` is tactical and should stay short.
- A coding agent is expected to drive every stage, not just implementation.
- Multi-session projects must explicitly record Nova command surfaces,
  authoritative docs, and proof commands.
- Architecture drafts and implementation plans must pass an AI critique gate
  before human feedback is requested or acted on.
- Validation must prove acceptance criteria, not just list directories of tests.
- If the artifacts drift from reality, update them before continuing.

## Open These References As Needed

- `../references/nova-delivery-checks.md`
- `references/stage-checklists.md`
- `references/project-template.md`
- `references/plan-template.md`
- `references/status-template.md`
