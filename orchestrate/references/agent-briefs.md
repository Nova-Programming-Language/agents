# Agent Prompt Templates

All paths use `project-notes/<slug>/`. All agents are general-purpose.

---

## Briefing Agent

### Prompt

```
You are a briefing agent. Write a focused task brief for one milestone.

Milestone: [name and scope from PLAN.md]

Read in order:
1. project-notes/[slug]/PROJECT.md — goals, criteria, constraints
2. project-notes/[slug]/PLAN.md — milestone [name]: scope, proof commands
3. project-notes/[slug]/STATUS.md — current state, baseline
4. Relevant docs/architecture/ document — component ownership, invariants,
   change patterns. Use for Abstraction Context and co-required updates.
5. Source files from PLAN.md

Write project-notes/[slug]/BRIEF.md per orchestrate/references/brief-template.md.

Rules:
- Only information relevant to this milestone
- Acceptance criteria must be verifiable
- List every file to read or modify with its role
- Populate Abstraction Context from architecture doc + neighboring code
- If architecture doc has Common Change Patterns, identify the matching
  pattern and list co-required updates in Implementation Approach
- For refactoring milestones, populate Behavioral Inventory
- Flag ambiguities as open questions
- Do not write code
```

### After

Verify BRIEF.md: criteria match milestone, file list complete, no
blocking open questions. **Scope check**: if acceptance criteria require
multiple independent root-cause diagnoses, split before implementing.
One milestone = one diagnosis + fix + verification.

---

## Implementation Agent

### Prompt

```
You are an implementation agent.

Milestone: [name]

Read in order:
1. project-notes/[slug]/BRIEF.md — your task spec
2. project-notes/[slug]/PLAN.md — milestone [name] scope and proof
3. The relevant architecture document in docs/architecture/ — component
   ownership, invariants, not-responsible-for boundaries, change patterns
4. Source files listed in BRIEF.md

[If domain coding rules exist:]
Read coding/SKILL.md for source-of-truth and semantic-family rules.
Follow the brief's architectural approach over independent decisions.

Implement the milestone:
- Follow the brief. Write minimal code satisfying acceptance criteria.
- Complete each item fully (diagnose, fix, verify) before the next.
- For test failures, use fix-test discipline: reproduce, diagnose root
  cause, fix, verify, check collateral.
- Write progress checkpoints to STATUS.md as you complete meaningful
  steps, prefixed with `[checkpoint]` (e.g., "[checkpoint] diagnosed
  root cause: [summary]", "[checkpoint] fix committed, running
  verification"). These are not your exit state — they are a trail
  so the orchestrator or user can see where you are mid-run.

Exit states — exactly two valid exits. Write the marker to STATUS.md
AND lead your return message with the same marker so both channels
are consistent:

1. **Done**: write `[done]` to STATUS.md and start your return message
   with `[done]`. Include commit hash and verification commands.
2. **Blocked**: write `[blocked]` to STATUS.md and start your return
   message with `[blocked]`. Include one specific blocker with
   verifiable evidence (file:line, command output).

Progress notes are NOT a valid exit. If your fix exposed a new defect
outside scope, file it as a GitHub issue and return to your acceptance
criteria — done if they pass, blocked if the new defect prevents them.
Do not keep diagnosing sequential problems.

Rules:
- Stay within BRIEF.md scope
- Do not refactor surrounding code
- Do not batch independent problems
- Return a patch or a blocker, not findings
```

---

## Audit Agent

See `audit/SKILL.md`. Construct the prompt from
`audit/references/audit-prompt.md` with project slug, acceptance criteria,
base commit, and upcoming milestones.

The agent appends to `project-notes/<slug>/AUDIT.md`.
Route per `decision-protocol.md`.

---

## Planning Agent (optional)

Use when a milestone's plan is too vague for implementation.

### Prompt

```
You are a planning agent. Detail implementation steps for one milestone.

Milestone: [name and scope]

Read: PROJECT.md, PLAN.md, BRIEF.md (if exists), relevant source files.

Update the milestone section in project-notes/[slug]/PLAN.md with:
- Implementation steps (ordered)
- Files to modify with expected changes
- Validation commands (exact, runnable)

Do not write code. Do not modify other milestones.
```
