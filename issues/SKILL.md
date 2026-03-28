---
name: issues
description: Report and fix bugs via GitHub issues; use when filing new issues from findings, picking up existing issues to fix, or triaging issue backlogs. Covers the full lifecycle from discovery through verified close.
---

# Issues

## When to Use

- Filing a bug or defect as a GitHub issue
- Picking up an existing issue to fix
- Triaging a set of issues
- Closing issues with verified evidence

Uses `gh` CLI for all GitHub operations.

## Report: Filing Issues

### From a finding

When you discover a bug, audit finding, or regression failure that
should be tracked:

1. **Verify first.** Reproduce the issue with an exact command. Do not
   file from memory or assumption. Use `bug-report` investigation
   discipline if the reproduction is non-trivial.

2. **Create the issue** using `references/issue-template.md` for
   structure. File with `gh issue create`.

3. **Label it.** Apply labels for subsystem, severity, and source:
   - Subsystem: `frontend`, `interpreter`, `c-backend`, `tests`,
     `cli`, `runtime`, `docs`
   - Severity: `bug`, `enhancement`, `blocker`
   - Source: `audit`, `regression`, `manual` (how it was found)

4. **Link context.** If the issue came from an audit, regression run,
   or project milestone, reference it in the issue body.

### From audit results

When an `audit` produces findings that need tracking beyond the current
session:

- Each actionable finding becomes one issue
- Copy the finding's evidence (file:line, commands) into the issue
- Reference the AUDIT.md section date

### From regression results

When a regression check produces unexpected failures:

- One issue per distinct root cause (not per failing test — multiple
  tests may fail from the same cause)
- Include the regression command, baseline, and current results
- Include the structured failure data if available

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
- The fix (one sentence)
- The verification command and its output

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

Multiple agents may share a single GitHub account, so assignment
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
   gh issue edit <number> --remove-label "c-backend" --add-label "frontend"
   ```
2. Comment with the diagnosis so the next agent doesn't repeat it:
   ```
   gh issue comment <number> --body "🔀 Reassigned from c-backend to frontend by session <session-id>.

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

## Triage: Working Through a Backlog

When presented with multiple issues:

1. Read all issue titles and labels to understand the landscape.
2. Group by likely root cause — multiple issues may share one.
3. Prioritize: blockers first, then bugs, then enhancements.
4. Claim and fix one at a time using fix-test discipline. Do not batch.
5. After each fix, check if other issues in the same group are
   now resolved. Close them with evidence if so.

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
