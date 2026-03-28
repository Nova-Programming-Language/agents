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

## Triage: Working Through a Backlog

When presented with multiple issues:

1. Read all issue titles and labels to understand the landscape.
2. Group by likely root cause — multiple issues may share one.
3. Prioritize: blockers first, then bugs, then enhancements.
4. Fix one at a time using fix-test discipline. Do not batch.
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
