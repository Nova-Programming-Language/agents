---
name: issues
description: Report and fix bugs via the project's issue tracker (e.g. GitHub issues) or as verified repo-local bug reports; use when filing issues from findings, writing a bug report with exact reproduction and ownership clues, picking up issues to fix, or triaging backlogs. Covers discovery through verified close.
---

# Issues

## When to Use

- Filing a bug or defect as an issue in the project's tracker
- Writing a verified, repo-local bug report (see Local Bug Report)
- Picking up an existing issue to fix
- Triaging a set of issues
- Closing issues with verified evidence

The tracker, its CLI, and its label set are in profile §Repository (see
`../references/project-profile.md`). The command examples below use GitHub's
`gh` CLI; translate them for another tracker.

## Report: Filing Issues

### From a finding

When you discover a bug, audit finding, or regression failure that
should be tracked:

1. **Verify first.** Reproduce the issue with an exact command. Do not
   file from memory or assumption. If the reproduction is non-trivial,
   investigate it per Local Bug Report first.

2. **Create the issue** using `references/issue-template.md` for
   structure. File with the tracker's create command (`gh issue create` on
   GitHub).

3. **Assign to a component.** Every issue must have exactly one
   subsystem label. This is how agents find work and how triage
   routes issues. Apply the label based on where the defect is,
   not where the symptom appears:

   The subsystem labels, and which kinds of failure map to which label,
   are in profile §Repository. To decide the correct component:
   - If the stack trace or error points to a specific module or package,
     use the component that owns it (the architecture map, profile §Paths)
   - Otherwise apply the profile's failure-kind → label rules
   - If uncertain, make your best hypothesis and note it in the issue
     body — triage will correct it if wrong

   Also apply a severity label (bug, enhancement, blocker) and a source
   label (audit, regression, manual), using the project's names for them
   from profile §Repository.

   ```
   gh issue create --label "<subsystem>" --label "<severity>" --label "<source>" ...
   ```

4. **Link context.** If the issue came from an audit, regression run,
   or project milestone, reference it in the issue body.

### Local Bug Report

Write a durable report in the reports directory (profile §Paths) when the
reproduction is non-trivial and needs structured investigation before filing,
or when a repo-local record of a defect or blocker is wanted. It is not for
transient debug notes.

1. Reproduce with the narrowest command. Rerun after a clean rebuild when
   relevant, and capture the exact failing output or behavior.
2. Record the exact command or input file, and the branch or commit under
   test: the current default branch (profile §Repository), a feature branch,
   or a dirty worktree.
3. Record whether a clean rebuild was performed, and state explicitly if it
   changes the outcome. Record whether the bug still reproduces on the current
   default branch.
4. Distinguish "originally observed" from "currently reproducible". If it no
   longer reproduces, say so plainly and record what changed during
   verification.
5. Name the most likely subsystem (the architecture map, profile §Paths), not
   a person, and say why.
6. If it blocks another task, say which commands or workflows are blocked.
7. Write it per `references/bug-report-template.md`, named by the reports
   directory's pattern (typically a date plus a short slug), using the current
   local date, not a guessed one.
8. Keep it factual: exact commands, file paths, and observed errors; no
   speculation that is not tied to evidence.

### From audit results

When an `audit` produces findings that need tracking beyond the current
session:

- Each actionable finding becomes one issue
- Copy the finding's evidence (file:line, commands) into the issue
- Reference the AUDIT.md section date

### From regression results

When a regression check produces unexpected failures:

- Get the structured failure records from the regression tooling
  (profile §Regression)
- Group records by their owner/phase fields to identify distinct root causes
  — multiple tests may fail from the same cause
- One issue per distinct root cause, not per failing test
- Include the regression command, baseline, and current results
- Paste the relevant structured failure record(s) into the issue body so the
  fixer has machine-readable context from the start

### What makes a good issue

- **Title**: the symptom in one line, specific enough to find later
- **Reproduction**: exact command that demonstrates the problem
- **Expected vs actual**: what should happen vs what does happen
- **Evidence**: file:line references, error output, test names
- **Ownership hypothesis**: which subsystem likely owns this

What does NOT make a good issue:
- Vague titles ("something is broken")
- No reproduction steps
- Speculation without evidence
- Batched issues ("fix these 5 things" — one issue per defect)

## Fix: Resolving Issues

### Picking up an issue

1. Read the issue fully: title, body, comments, labels.
2. Reproduce the problem using the issue's reproduction steps.
   If it cannot be reproduced, comment on the issue with what you
   tried and stop.
3. Use `fix-test` discipline for the fix:
   - Diagnose root cause
   - Fix the root cause (not the symptom)
   - Verify the fix
   - Check for collateral damage

### Closing an issue

Close with evidence, not just "I think I fixed it":

```
gh issue close <number> --comment "Fixed in <commit>.

Root cause: <one sentence>
Fix: <one sentence>
Verified: <exact command and result>"
```

Every close comment must include:
- The commit hash
- The root cause (one sentence)
- The fix (one sentence), and which sites of the violated rule it covers
- The verification command and its output

Use "Fixes #N" only when every site of the class is covered; otherwise
"refs #N", and leave the issue open listing the uncovered sites.

Do not close without verification. If the fix is committed but not yet
verified in a broader regression, say so in the comment and leave the
issue open until verified.

### Issues that are blockers

If an issue blocks another component or milestone:

- Label it `blocker`
- Reference what it blocks (milestone, other issue, component)
- If it's blocking an orchestrate pipeline, record it in STATUS.md
  under Blocked with the issue URL

## Claiming: Preventing Duplicate Work

Multiple agents may share a single tracker account, so assignment
cannot distinguish between them. Claims use comments with a unique
session identifier instead.

### Before picking up an issue

Check the issue's recent comments for an active claim:

```
gh issue view <number> --json comments --jq '.comments[-5:][].body'
```

If any recent comment starts with `🔒 Claimed by session`, the issue
is taken. Skip it and pick another.

### Claiming

Comment with your session ID before doing any work:

```
gh issue comment <number> --body "🔒 Claimed by session <session-id> — starting diagnosis."
```

The session ID should be unique per agent invocation (e.g., a
timestamp, UUID, or the conversation/process identifier). The claim
comment is the lock — do not read code, reproduce, or diagnose
before posting it.

### Releasing

If you cannot fix the issue (blocker, wrong subsystem, need more
context), release it:

```
gh issue comment <number> --body "🔓 Released by session <session-id> — [reason]."
```

A released issue is available for other agents to claim.

### Reassigning to a different component

If diagnosis reveals the root cause is in a different subsystem than
the issue's current label:

1. Relabel the issue to the correct subsystem:
   ```
   gh issue edit <number> --remove-label "<old-subsystem>" --add-label "<new-subsystem>"
   ```
2. Comment with the diagnosis so the next agent doesn't repeat it:
   ```
   gh issue comment <number> --body "🔀 Reassigned from <old-subsystem> to <new-subsystem> by session <session-id>.

   Diagnosis: [root cause summary]
   Evidence: [file:line, command output]
   The defect is in [component] because [reason]."
   ```
3. If you can fix the other component, keep your claim and fix it.
4. If you cannot (wrong expertise, different worktree, blocked),
   release the claim. The diagnosis comment stays so the next agent
   picks up where you left off.

Do not leave the old label in place "just in case." The label should
reflect the current understanding of ownership.

### Stale claims

If an issue has a claim comment but no follow-up activity (no
further comments, no referencing commits), the claim may be stale —
the agent may have crashed or the session may have ended. Agents
do not override other agents' claims. Only the user decides whether
to release a stale claim.

## Triage: Component Assignment and Prioritization

Triage has two jobs: make sure every issue is assigned to the right
component, and prioritize the backlog for fixing.

### Component verification pass

Before fixing anything, verify component labels are correct:

```
gh issue list --state open --json number,title,labels
```

For each issue:
- Does it have exactly one subsystem label? If zero, read the issue
  and assign one. If multiple, pick the one where the defect lives
  (not where the symptom appears) and remove the others.
- Does the label look correct based on the title and body? A quick
  read is enough — full diagnosis happens during fixing, not triage.
- If you can tell the label is wrong from the title alone (e.g., a
  code-generator panic labeled with the parser's subsystem), relabel it now.

This pass is fast — you are reading titles and labels, not code.
Comment on any relabeled issue with your reasoning.

### Picking issues by component

When an agent is working a specific component, filter:

```
gh issue list --state open --label "<subsystem>" --json number,title,labels
```

Only claim issues labeled for your component. If during diagnosis you
discover the root cause is in a different component, follow the
reassignment workflow (see Reassigning to a different component).

### Prioritizing within a component

1. blocker first — these block other work
2. bug + regression — regressions from recent changes
3. bug — other defects
4. enhancement — last

Within the same priority, prefer issues with more evidence (exact
reproduction, file:line references) — they are faster to fix.

### Fixing

Claim and fix one at a time using `fix-test` discipline. Do not batch.
After each fix, check if other issues in the same component are now
resolved. Close them with evidence if so.

## Integration

- **With orchestrate**: Issues can be milestones. The orchestrator
  creates a brief per issue, the implementation agent fixes it, the
  audit verifies it, and the issue is closed on commit.
- **With project-creation**: Link issues to PROJECT.md acceptance
  criteria or PLAN.md milestones.
- **With regressions**: Unexpected regressions become issues. Declared
  regressions reference the resolving milestone, not an issue.

## References

- `references/issue-template.md` — issue body structure
- `references/bug-report-template.md` — repo-local bug report structure
- `../references/project-profile.md` — profile §Repository holds the tracker
  and labels
