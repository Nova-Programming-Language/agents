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

Preserve the approved milestone as the unit of delivery. Do not progressively
thin-slice it into `a`/`b`, pilot, cleanup, or follow-up phases. A split
requires measured independent root causes or independently auditable and
releasable outcomes; elapsed time, file count, context size, and a failing
test are insufficient.

### 0. Scout (agent, optional — measure before ratifying)
Spawn a measurement-only agent (`references/agent-briefs.md`) BEFORE
briefing when the milestone's premises rest on implicit complex
assumptions — anything the plan asserts about what the system can
distinguish, express, or support that has not been directly measured
(e.g. "the analysis can discriminate X from Y", "this fact is available
at that point", "this fence contains the fix"). The scout writes no
production code and commits no fixtures; it answers the load-bearing
questions with measurements recorded durably (STATUS.md), and the brief
is then written FROM those facts.

Warranted when: the change is verdict/behavior-shifting; the mechanism
lives in a substrate no one has probed; a previous sub-step was
falsified at implementation time; or the brief-writer would otherwise
have to hypothesize a discriminating fact. Skip for mechanical work
whose premises are already established.

Never ratify a scope fence whose discriminating facts have not been
measured to exist inside it.

### 1. Brief (agent)
Spawn a briefing agent (`references/agent-briefs.md`). It reads project
artifacts and source files, then writes `project-notes/<slug>/BRIEF.md`.
If a scout ran, the brief must encode the scout's measured facts — a
brief premise that contradicts or exceeds the measurements is a defect.

### 2. Implement (agent)
Spawn an implementation agent **in foreground** and wait for it to
complete. Do not proceed until the agent has returned its final message.

**`[checkpoint]` entries in STATUS.md do NOT mean the agent is done.**
They are mid-run progress written while the agent is still working.
Seeing a checkpoint is not a reason to stop waiting — it is confirmation
the agent is still making progress. Only proceed to step 2a after the
agent has returned its final message to you.

### 2a. Check for completion (orchestrator)
The agent has returned. Now read STATUS.md for the final marker:
- `[done]` — patch committed and verified. Proceed to audit.
- `[blocked]` — one specific blocker. Route per decision protocol.
- No `[done]` or `[blocked]`, only `[checkpoint]` entries — the agent
  exited without completing. Route per the **Incomplete** section of
  `references/decision-protocol.md`.

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
After commit: run the validation gate assigned by PLAN.md, check plan
coherence, and check cumulative progress. Do not run canonical/full regression
after every file or checkpoint; reserve it for defined production-changing
milestone gates and final release.
See `references/decision-protocol.md`.

## Orchestrator Discipline

The main context does ONLY: read artifacts, construct agent prompts,
spawn agents, read results from disk, make pass/escalate decisions,
update STATUS.md and LOG.md, and own long-running verification (below).

The main context does NOT: read source code, write code, run tests, or
debug. Plan coherence uses AUDIT.md summaries, not source code.
Exception: long-running verification commands are the orchestrator's to
run (as session background tasks), because sub-agents cannot host them
reliably — see Long-Running Verification.

### Delegated Package Onboarding

For any worker touching or validating a package repository, read
`../references/package-onboarding.md` and include its exact applicable files
in the worker brief. A generic instruction to "follow repository rules" is not
enough. Require the worker to report which READMEs it read and whether the
commands matched package-specific prerequisites. Audit already-started work
against those READMEs before accepting it.

## Long-Running Verification

A sub-agent's processes — including anything it started with a
background bash call — are killed the moment the agent returns its
final message. Only background tasks started by the MAIN session
survive turn boundaries and emit completion notifications. Two rules
follow:

1. **Workers must never "start and wait" on their own background.** An
   implementation agent that needs a long verification (full
   regression, whole-repo verification) runs it in the FOREGROUND and
   stays alive until it completes. A worker that backgrounds the run
   and returns "waiting for results" has orphaned a dead process and
   made an invalid exit. Put this rule in the implementation brief for
   any milestone whose proof includes a long-running command.
2. **Premature exit during verification is recoverable without
   re-briefing.** If a worker returns with only `[checkpoint]` entries
   and its last checkpoint says verification was started or pending:
   do NOT re-run the brief and do NOT spawn a continuation agent (a
   new Agent call does not inherit the returned worker's context, and
   the second worker will hit the same wall). Instead the orchestrator
   (a) re-runs the verification command itself as a main-session
   background task, (b) on success performs the commit exactly as the
   brief specified — the diff is already in the working tree and the
   worker's checkpoints record its verification claims — and (c)
   records the outcome in STATUS.md on the worker's behalf, then
   proceeds to audit. On verification failure, escalate per the
   decision protocol with the failing output.

When sequencing allows it, prefer the split that avoids the problem:
worker does implementation + fast targeted tests in foreground and
records its claims; orchestrator owns the heavyweight verification gate
before audit/commit.

This is a validation-ownership split, not a new implementation phase.

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

## References

- `../references/package-onboarding.md` — mandatory delegated onboarding for
  package-repository work
