---
name: orchestrate
description: Drive a multi-milestone project by delegating brief, implement and audit work to agents; use when project-creation artifacts exist. Not for setting up a project (use project-creation).
---

# Orchestrate

## When to Use

Use when a session spans several milestones and work is delegated to agents
while the main context stays goal-focused. Requires the `project-creation`
artifacts under `project-notes/<slug>/`. Reasons behind the rules below are in
`references/rationale.md`.

## The Pipeline

For each incomplete milestone in PLAN.md. Preserve the approved milestone as
the unit of delivery; split one only per `references/brief-template.md`,
"Scope validation" and "Size by risk and reward".

### 0. Scout (agent, optional — measure before ratifying)
- Spawn a measurement-only scout (`references/agent-briefs.md`) before
  briefing when the milestone's premises rest on implicit complex assumptions
  that have not been measured: what the system can distinguish, express, or
  support (e.g. "the analysis can discriminate X from Y", "this fact is
  available at that point", "this fence contains the fix").
- It is warranted when the change shifts a verdict or behavior, the mechanism
  lives in a substrate no one has probed, a previous sub-step was falsified at
  implementation time, or the brief-writer would otherwise hypothesize a
  discriminating fact. Skip it for mechanical work whose premises are
  established.
- The scout writes no production code and commits no fixtures; it records its
  measurements in STATUS.md, and the brief is written from them.
- Never ratify a scope fence whose discriminating facts have not been measured
  to exist inside it.
- **Design is required** when a milestone implements a rule that changes
  run-time values, or that must be enforced in more than one engine or entry
  point. The design states the representation, the single home of the rule,
  and the internal error raised when it is bypassed, and it gets a `critique`
  critique before the brief. Never write a brief from approved rule text
  alone.

### 1. Brief (agent)
Spawn a briefing agent (`references/agent-briefs.md`); it writes
`project-notes/<slug>/BRIEF.md`. If a scout ran, the brief encodes its
measured facts; a premise that contradicts or exceeds them is a defect.

### 2. Implement (agent)
Spawn an implementation agent **in the foreground** and wait for its final
message. `[checkpoint]` entries in STATUS.md do not mean it is done; they are
progress while it works.

### 2a. Check for completion (orchestrator)
After the agent returns, read STATUS.md for the final marker:
- `[done]` — implemented and verified, not yet committed (the commit is
  step 5). Proceed to audit.
- `[blocked]` — one specific blocker. Route per the decision protocol.
- Only `[checkpoint]` entries — the agent exited without completing. Route
  per **Incomplete** in `references/decision-protocol.md`.

### 3. Audit (agent)
Spawn an audit agent per the `audit` skill; it appends a dated section to
`project-notes/<slug>/AUDIT.md`.

### 4. Evaluate (orchestrator)
Read the latest AUDIT.md section. Attribute any failure per
`../references/failure-attribution.md`, then route per
`references/decision-protocol.md`:
- **Pass** — commit.
- **Fail** — escalate to the user with the audit findings. No automatic
  retries; the user decides next steps.
- **Incomplete** — the worker returned findings instead of a patch. Split and
  re-brief per the decision protocol.

### 5. Commit (orchestrator)
After a passing audit, commit with `checkin`.

### 6. Between milestones (orchestrator)
Run the validation gate PLAN.md assigns, check plan coherence, and check
cumulative progress, per `references/decision-protocol.md`. Choose which tests
to run per `../regressions/SKILL.md`, "When to Run What".

## Orchestrator Discipline

- The main context only reads artifacts, writes agent prompts, spawns agents,
  reads results from disk, makes pass/escalate decisions, updates STATUS.md
  and LOG.md, and runs long-running verification (below).
- It does not read source code, write code, run tests, or debug. Plan
  coherence uses AUDIT.md summaries, not source code.
- Rule texts are frozen while a step is open. A new ruling waits for the next
  step boundary unless it contradicts the step in flight; then the step
  returns to design.
- Never implement one rule on two parallel branches; merge first.
- Read outcomes from the artifact files (STATUS.md, AUDIT.md, BRIEF.md), not
  from agent return messages. The markers `[done]`, `[blocked]`, and
  `[checkpoint]` are the authoritative signal.

### Choosing the Agent's Model

Pick each spawned agent's model yourself; don't ask the user. Pass `model` on
the Agent call:

| Work | Model |
|---|---|
| Audits (step 3), class-coverage audits, critique critiques of spec drafts and plans | `fable` |
| Milestone briefs (step 1), and so also their critique critiques | `fable` |
| Implementation (step 2), scouts, debugging, spec drafting | `opus` (or inherit) |
| Mechanical work: filing issues from written notes, repro sweeps, disk/build/state checks, copying results | `sonnet` |

Forks (`subagent_type: "fork"`) always run on the orchestrator's model. A
running agent keeps its model, so a change applies only to agents spawned
after it. If an agent on the cheaper tier misses something, or an audit
underperforms, change the mix and record it in LOG.md.

### Delegated Package Onboarding

For any worker touching or validating a package repository, brief it and
accept its result per `../references/package-onboarding.md`, "Delegated work".

## Attributing Failures

Attribute every failure before routing it, per
`../references/failure-attribution.md`. Two obligations are the orchestrator's:

- **Milestones carry entry and exit gates.** Start a milestone by running the
  prior milestones' suites and recording the counts in STATUS.md. Close it by
  passing its own suite with every prior suite still green at a recorded
  commit. Never start a milestone on a red baseline; escalate instead.
- **The fix site is not negotiable by convenience.** Do not accept a patch
  that compensates downstream for a lower layer's or another package's
  defect; route it per that reference, "When Another Component Owns It". An
  audit that passed such a patch has not passed (`../audit/SKILL.md`).

## Long-Running Verification

A sub-agent's processes, background ones included, die when it returns; only
main-session background tasks survive.

1. **Workers never start a long verification and return.** A worker runs a
   long verification (full regression, whole-repo verification) in the
   foreground and stays alive until it finishes. Put this rule in the
   implementation brief of any milestone whose proof includes a long command.
2. **Recover a premature exit during verification without re-briefing.** If a
   worker returns with only `[checkpoint]` entries and the last one says
   verification was started or pending, do not re-run the brief and do not
   spawn a continuation agent. Instead: (a) re-run the verification yourself
   as a main-session background task; (b) on success, record the outcome in
   STATUS.md on the worker's behalf; (c) proceed to audit, then commit at
   step 5. On failure, escalate per the decision
   protocol with the failing output.

When sequencing allows, prefer the split that avoids the problem: the worker
does implementation and fast targeted tests in the foreground and records its
claims; the orchestrator owns the heavyweight gate, after the audit passes and
before the commit.
This is a split of validation ownership, not a new implementation phase.

## Artifacts

All in `project-notes/<slug>/`:

| Artifact | Written by | Lifecycle |
|---|---|---|
| PROJECT.md | project-creation | durable |
| PLAN.md | project-creation / orchestrator | durable |
| STATUS.md | all agents / orchestrator | point-in-time — REPLACED, never appended |
| LOG.md | orchestrator | append-only history |
| AUDIT.md | audit agent | append-only |
| BRIEF.md | briefing agent | per-milestone |

Keep STATUS.md (state) and LOG.md (history) per
`../project-creation/references/status-template.md`, "Keeping STATUS.md
current", and fix any breach of its defect list before continuing.

## Session Start
1. List `project-notes/` to identify the active slug.
2. Read PROJECT.md, PLAN.md, STATUS.md.
3. If AUDIT.md exists, read the latest section.
4. Enter the pipeline at the next incomplete milestone.

## Session End
1. Rewrite STATUS.md and append to LOG.md per
   `../project-creation/references/status-template.md`, "Keeping STATUS.md
   current".
2. Verify: STATUS.md has one state, LOG.md has the history, and no superseded
   text exists only in STATUS.md.

## References

- `../references/project-profile.md` — where project commands, paths, and
  conventions live (cited as "profile §Section")
- `../references/package-onboarding.md` — delegated onboarding for package work
- `../references/failure-attribution.md` — which component owns a defect and
  which milestone owns the work
- `references/agent-briefs.md`, `references/brief-template.md`,
  `references/decision-protocol.md` — prompts, brief format, routing
- `references/rationale.md` — why the pipeline is shaped this way
