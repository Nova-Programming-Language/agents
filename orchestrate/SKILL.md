---
name: orchestrate
description: Drive long sessions via agent pipeline; use when a project has delivery artifacts and work should be delegated to focused agents rather than done inline, keeping the main context as a thin orchestrator that preserves goal alignment throughout.
---

# Orchestrate

## When to Use

Use when a session will span multiple milestones and you want to preserve
goal alignment throughout. The main context becomes a pure orchestrator —
all substantive work happens in agents that read from and write to disk.

## Prerequisites

Delivery artifacts must exist under `project-notes/<slug>/`:

- `project-notes/<slug>/PROJECT.md`
- `project-notes/<slug>/PLAN.md`
- `project-notes/<slug>/STATUS.md`

If missing, use the `delivery` skill to create them first.
Identify the `<slug>` by listing `project-notes/`.

## Read First

- `project-notes/<slug>/PROJECT.md`
- `project-notes/<slug>/PLAN.md`
- `project-notes/<slug>/STATUS.md`
- `references/agent-briefs.md`

## The Loop

Repeat for each incomplete milestone in PLAN.md:

### 1. Brief (agent)

Spawn a **briefing agent** using the prompt in `references/agent-briefs.md`.
Wait for it to write `project-notes/<slug>/BRIEF.md`.
Read the brief. Confirm it matches the milestone's intent.

### 2. Implement (agent)

Spawn an **implementation agent** using the prompt in `references/agent-briefs.md`.
The agent reads BRIEF.md and PLAN.md, implements the milestone, and updates
STATUS.md. If a prior attempt failed, append the failure context from the
latest AUDIT.md section to the agent prompt.

### 3. Audit (agent)

Spawn an **audit agent** using the `audit` skill. See
`audit/references/audit-prompt.md` for the prompt template and
`references/agent-briefs.md` for how to invoke it from the pipeline.
The agent reads BRIEF.md, runs `git diff`, evaluates functional
correctness, structural quality, and forward impact, then appends a
dated section to `project-notes/<slug>/AUDIT.md`.

### 4. Evaluate (orchestrator)

Read the latest section in AUDIT.md and route per
`references/decision-protocol.md`:

- **Pass** — commit the milestone, update STATUS.md, append to LOG.md,
  move to next milestone.
- **Fail** — re-launch step 2 with failure context. Max 2 retries, then
  escalate to user.
- **Ambiguous** — escalate to user with specific questions.

### 5. Commit (orchestrator)

After a milestone passes all three audit dimensions, commit using the
`checkin` skill. The commit message should name the milestone and
summarize what was done. This creates a clean base commit for the next
milestone's audit diff.

Do not commit partial or failing milestones. The commit is the gate
between milestones — it makes the passing state permanent and gives the
next audit agent a clean `git diff [base]..HEAD`.

### 6. Between milestones (orchestrator)

After commit, before starting the next milestone:

1. **Regression check** — run the project's test/regression suite to
   verify the commit did not break prior milestones.
2. **Plan coherence** — compare what was implemented against what was
   planned. Update PLAN.md if the implementation diverged.
3. **Cumulative progress** — re-read PROJECT.md acceptance criteria.
   Verify remaining milestones still cover all criteria. Flag gaps.

See `references/decision-protocol.md` for details. Do not start the
next milestone until all three checks are clean.

## Orchestrator Discipline

The main context does ONLY:

1. Read project artifacts from disk
2. Construct agent prompts from `references/agent-briefs.md`
3. Spawn agents and read their results from disk
4. Make pass/fail/escalate decisions
5. Update STATUS.md and LOG.md

The main context does NOT:

- Read source code
- Write or modify code
- Run tests or builds
- Debug failures

All substantive work is delegated to agents. This keeps the orchestrator's
context small and goal-focused regardless of session length.

## Artifact Roles

All artifacts live in `project-notes/<slug>/`.

| Artifact | Written by | Read by | Lifecycle |
|---|---|---|---|
| PROJECT.md | delivery / orchestrator | all agents | durable |
| PLAN.md | delivery / orchestrator | all agents | durable |
| STATUS.md | all agents / orchestrator | all agents | durable |
| LOG.md | orchestrator | orchestrator | append-only |
| AUDIT.md | audit agent | orchestrator | append-only (dated sections) |
| BRIEF.md | briefing agent | impl + audit agents | per-milestone |

Agents communicate through these files, not through return values.
Large outputs (diffs, test results, debug traces) stay in the agent's
context or on disk — they never flow through the orchestrator.

## Session Start

1. List `project-notes/` to identify the active project slug.
2. Read PROJECT.md, PLAN.md, STATUS.md.
3. If AUDIT.md exists, read the latest section — the last pipeline may
   have been interrupted.
4. Identify the next incomplete milestone.
5. Enter the loop.

## Session End

1. Update STATUS.md with current state and next action.
2. Append dated entry to LOG.md.
3. Ensure all project artifacts reflect reality.
