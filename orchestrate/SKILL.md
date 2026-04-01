---
name: orchestrate
description: Drive long sessions via agent pipeline; use when a project has artifacts from project-creation and work should be delegated to focused agents, keeping the main context as a thin orchestrator.
---

# Orchestrate

## When to Use

Use when a session will span multiple milestones and you want to delegate
work to agents while keeping the main context goal-focused.

Requires artifacts from `project-creation` under `project-notes/<slug>/`.

## The Pipeline

For each incomplete milestone in PLAN.md:

### 1. Brief (agent)
Spawn a briefing agent (`references/agent-briefs.md`). It reads project
artifacts and source files, then writes `project-notes/<slug>/BRIEF.md`.

### 2. Implement (agent)
Spawn an implementation agent. It reads BRIEF.md and PLAN.md, implements
the milestone, and updates STATUS.md. The agent must exit with one of
two states: **done** (patch + verification) or **blocked** (one specific
blocker with evidence). Progress notes without a patch are not a valid
exit — see `references/decision-protocol.md` for handling.

### 2a. Check for completion (orchestrator)
Before spawning the audit, read STATUS.md and check the final status
marker. The implementation agent writes progress checkpoints during
work (prefixed `[checkpoint]`) and a final exit marker when done:
- `[done]` — patch committed and verified. Proceed to audit.
- `[blocked]` — one specific blocker. Route per decision protocol.
- No final marker, only checkpoints — the agent did not finish cleanly.
  Route per the **Incomplete** section of `references/decision-protocol.md`.

Do not treat `[checkpoint]` entries as the final state.

### 3. Audit (agent)
Spawn an audit agent per the `audit` skill. It evaluates functional
correctness, structural quality, and forward impact, then appends a
dated section to `project-notes/<slug>/AUDIT.md`.

### 4. Evaluate (orchestrator)
Read the latest AUDIT.md section. Route per `references/decision-protocol.md`:
- **Pass** — proceed to commit.
- **Fail** — escalate to user with the audit findings.
- **Incomplete** — the worker returned findings instead of a patch.
  Split and re-brief per the decision protocol.

No automatic retries. If the audit fails, the user decides next steps.

### 5. Commit (orchestrator)
After passing audit, commit using `checkin`. This creates a clean base
for the next milestone's audit diff.

### 6. Between milestones (orchestrator)
After commit: regression check, plan coherence, cumulative progress.
See `references/decision-protocol.md`.

## Orchestrator Discipline

The main context does ONLY: read artifacts, construct agent prompts,
spawn agents, read results from disk, make pass/escalate decisions,
update STATUS.md and LOG.md.

The main context does NOT: read source code, write code, run tests, or
debug. Plan coherence uses AUDIT.md summaries, not source code.

## Artifacts

All in `project-notes/<slug>/`:

| Artifact | Written by | Lifecycle |
|---|---|---|
| PROJECT.md | project-creation | durable |
| PLAN.md | project-creation / orchestrator | durable |
| STATUS.md | all agents / orchestrator | durable |
| LOG.md | orchestrator | append-only |
| AUDIT.md | audit agent | append-only |
| BRIEF.md | briefing agent | per-milestone |

**Authoritative state is in files, not agent return messages.** When a
sub-agent completes, it returns a text message — but that message is a
summary, not the source of truth. Always read the relevant artifact file
(STATUS.md, AUDIT.md, BRIEF.md) to determine the actual outcome. Agent
return messages may be truncated, ambiguous, or describe intermediate
progress. The file markers (`[done]`, `[blocked]`, `[checkpoint]`) are
the authoritative signal.

## Session Start
1. List `project-notes/` to identify the active slug.
2. Read PROJECT.md, PLAN.md, STATUS.md.
3. If AUDIT.md exists, read the latest section.
4. Enter the pipeline at the next incomplete milestone.

## Session End
1. Update STATUS.md.
2. Append to LOG.md — software work only, not agent workflow.
