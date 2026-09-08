---
name: project-creation
description: Create project artifacts for multi-session work; use when starting a new project that spans multiple commits or sessions and needs persistent context, stage gates, and required links.
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

- `docs/agent-rules.md`
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
| Architecture | `project-creation`, `design`, `review-nova` |
| Implementation Plan | `project-creation`, `review-nova` |
| Implementation | `coding` directly, or via `orchestrate` for agent-driven work |
| Validation | `regressions`, `building`; `debug-nova` when needed |
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

LOG.md records **software engineering work only** — code changes, design
decisions, milestones, technical blockers. Not agent workflow (spawning
agents, confirming commits, updating artifacts). The "next" field must
name a concrete software task.

## Rules

- PROJECT.md is the durable source of project truth.
- PLAN.md is the source of truth for milestone sequencing.
- STATUS.md is tactical and short.
- LOG.md is append-only. It is the ONLY place history accumulates.
- STATUS.md is point-in-time state, not a log. Rewrite it; never append a dated
  section to it. When a milestone closes, move its checkpoints to LOG.md and
  delete them from STATUS.md. If STATUS.md has grown past roughly 150 lines or
  carries superseded checkpoints, migrate before doing anything else — a status
  file nobody rewrites is where stale claims survive.
- Default to cohesive, independently auditable milestones. Do not split work
  because of elapsed time, file count, context size, or one newly failing test.
  A split requires independently releasable value or measured independent root
  causes. Do not invent `a`/`b`, pilot, cleanup, or follow-up phases without
  explicit plan approval.
- Broad public contracts require a completion matrix covering success and
  failure behavior, engines/surfaces, lifecycle, and packaging.
- Architecture drafts and plans must pass AI critique before human review.
- Validation must prove acceptance criteria with exact commands.
- If artifacts drift from reality, update them before continuing.

## Open These References As Needed

- `../references/nova-delivery-checks.md`
- `references/stage-checklists.md`
- `references/project-template.md`
- `references/plan-template.md`
- `references/status-template.md`
