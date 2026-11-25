---
name: project-creation
description: Create the project-notes artifacts (PROJECT, PLAN, STATUS, LOG) for multi-session work; use when starting a project that spans sessions. Not for running its milestones (use orchestrate).
---

# Project Creation

## When to Use

Use when starting a project that spans sessions. Creates the persistent
artifacts that `orchestrate`, `audit`, and other skills depend on.

## Artifacts

All artifacts live in `project-notes/<slug>/`:

| File | Role |
|---|---|
| PROJECT.md | Durable backbone: goals, criteria, constraints |
| PLAN.md | Milestones, sequencing, validation commands |
| STATUS.md | Current checkpoint, regression baseline, working state |
| LOG.md | Append-only session history |

PROJECT.md must link to PRD (or `N/A` with reason), architecture docs
(or `N/A`), and implementation plan (`PLAN.md`).

One active project per checkout. Archive previous work before starting
a new project.

## Read First

- the project profile (`../references/project-profile.md` says how to find it)
- the project's rules file (profile §Rules)
- `references/stage-checklists.md`
- `references/project-template.md`
- `references/plan-template.md`
- `references/status-template.md`

## Session Start

1. Read `project-notes/<slug>/PROJECT.md`.
2. Read `project-notes/<slug>/PLAN.md`.
3. Read `project-notes/<slug>/STATUS.md`.
4. Confirm the current lifecycle stage before coding.

## Stage Routing

| Stage | Required skills |
|---|---|
| Requirements | `project-creation`, `prd` when needed |
| Architecture | `project-creation`, `design`, `critique` |
| Implementation Plan | `project-creation`, `critique` |
| Implementation | `coding` directly, or via `orchestrate` for agent-driven work |
| Validation | `regressions`, `building`; `debug` when needed |
| Commit/push | `checkin` |

See `references/stage-checklists.md` for exit criteria per stage.

## Workflow

1. Create `project-notes/<slug>/` and seed artifacts from templates.
2. Identify the current stage; load its checklist.
3. Use the required skill(s) for that stage.
4. Update artifacts before ending the session or changing stage.

## LOG.md

Append a dated entry at session end:

```md
## YYYY-MM-DD
- stage: implementation
- focus: what was worked on
- outcome: what was accomplished
- blockers: any technical blockers
- next: the next coding or design step
```

What LOG.md records: `references/status-template.md`, "Keeping STATUS.md
current". The "next" field must name a concrete software task.

## Rules

- PROJECT.md is the durable source of project truth.
- PLAN.md is the source of truth for milestone sequencing.
- STATUS.md is tactical and short.
- LOG.md rules: `references/status-template.md`, "Keeping STATUS.md current".
- STATUS.md is point-in-time state, not a log: rewrite it, never append, and
  migrate it before anything else when it breaks the defect list in
  `references/status-template.md`, "Keeping STATUS.md current".
- Default to cohesive, independently auditable milestones; split one only per
  `../orchestrate/references/brief-template.md`, "Scope validation" and "Size
  by risk and reward", and never without explicit plan approval.
- Broad public contracts require a completion matrix covering success and
  failure behavior, engines/surfaces, lifecycle, and packaging.
- Architecture drafts and plans must pass AI critique before human review.
- Validation must prove acceptance criteria with exact commands.
- If artifacts drift from reality, update them before continuing.

## Open These References As Needed

- `../references/project-profile.md`; the project's delivery checks (profile
  §Delivery checks)
- `references/stage-checklists.md`
- `references/project-template.md`
- `references/plan-template.md`
- `references/status-template.md`
