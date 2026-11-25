---
name: issues
description: File, triage, fix and close tracker issues, and write verified local bug reports; use for findings to file, defects to document, or issues to pick up. Not for a list of failing tests (use fix-test).
---

# Issues

## When to Use

- Filing a bug or defect as an issue in the project's tracker
- Writing a verified, repo-local bug report (see Local Bug Report)
- Picking up an existing issue to fix
- Triaging a set of issues
- Closing issues with verified evidence

The tracker, its CLI, and its label set are in profile §Repository (see
`../references/project-profile.md`). The commands below use GitHub's `gh`;
translate them for another tracker. Reasons behind the rules are in
`references/rationale.md`.

## Report: Filing Issues

### From a finding

1. **Verify first.** Reproduce with an exact command; never file from memory
   or assumption. If the reproduction is non-trivial, investigate it per
   Local Bug Report first.
2. **Create the issue** with the body in `references/issue-template.md`, using
   the tracker's create command (`gh issue create`).
3. **Label it.** Exactly one subsystem label, chosen by where the defect is,
   not where the symptom appears:
   - a stack trace or error in a specific module or package → the component
     that owns it (the architecture map, profile §Paths);
   - otherwise the profile's failure-kind → label rules (profile §Repository);
   - if uncertain, your best hypothesis, noted in the body; triage corrects it
     if wrong.

   Also apply a severity label (bug, enhancement, blocker) and a source
   label (audit, regression, manual), using the project's names:
   ```
   gh issue create --label "<subsystem>" --label "<severity>" --label "<source>" ...
   ```
4. **Link context**: the audit, regression run, or milestone it came from.

### Local Bug Report

Write a durable report in the reports directory (profile §Paths) when the
reproduction is non-trivial and needs structured investigation before filing,
or when a repo-local record of a defect or blocker is wanted. It is not for
transient debug notes.

1. Do not write the report from memory alone. Reproduce with the narrowest
   command. Rerun after a clean rebuild when
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

For findings that need tracking beyond the session: one issue per actionable
finding, with its evidence (file:line, commands) copied in and the AUDIT.md
section date referenced.

### From regression results

For unexpected failures:

- Get the structured failure records (profile §Regression) and group them by
  owner and phase to find distinct root causes.
- File one issue per distinct root cause, not per failing test.
- Include the regression command, the baseline, and the current results, and
  paste the relevant structured records into the body.

### What makes a good issue

Follow the Notes in `references/issue-template.md`: a specific one-line
symptom title, an exact reproduction, expected versus actual, evidence, and an
ownership hypothesis. One defect per issue; no vague titles, missing
reproductions, or speculation without evidence.

## Fix: Resolving Issues

### Picking up an issue

1. Read it fully: title, body, comments, labels. Claim it (see Claiming)
   before any other work.
2. Reproduce it from its reproduction steps. If it does not reproduce,
   comment with what you tried and stop.
3. Fix it with `fix-test` discipline: diagnose the root cause, fix the root
   cause rather than the symptom, verify, and check for collateral damage.

### Closing an issue

Close only with evidence:

```
gh issue close <number> --comment "Fixed in <commit>.

Root cause: <one sentence>
Fix: <one sentence>
Verified: <exact command and result>"
```

- Every close comment includes the commit hash, the root cause (one
  sentence), the fix (one sentence) and which sites of the violated rule it
  covers, and the verification command with its output.
- Use "Fixes #N" only when every site of the class is covered; otherwise use
  "refs #N" and leave the issue open listing the uncovered sites.
- Do not close without verification. If the fix is committed but a broader
  regression is still pending, say so in a comment and leave the issue open.

### Issues that are blockers

Label it `blocker`, reference what it blocks (milestone, issue, component),
and, in an orchestrate pipeline, record it in STATUS.md under Blocked with
the issue URL.

## Claiming: Preventing Duplicate Work

Claims are comments carrying a unique session identifier, because agents may
share one tracker account.

- **Before picking up**: check recent comments for an active claim. If one
  starts with `🔒 Claimed by session`, skip the issue.
  ```
  gh issue view <number> --json comments --jq '.comments[-5:][].body'
  ```
- **Claim** with a session ID unique to this agent run (timestamp, UUID, or
  process ID) before any work; do not read code, reproduce, or diagnose first.
  ```
  gh issue comment <number> --body "🔒 Claimed by session <session-id> — starting diagnosis."
  ```
- **Release** when you cannot fix it (blocker, wrong subsystem, missing
  context); the issue is then free for others.
  ```
  gh issue comment <number> --body "🔓 Released by session <session-id> — [reason]."
  ```
- **Reassign** when diagnosis puts the root cause in another subsystem:
  1. Relabel it; never leave the old label in place "just in case":
     ```
     gh issue edit <number> --remove-label "<old-subsystem>" --add-label "<new-subsystem>"
     ```
  2. Comment the diagnosis so the next agent doesn't repeat it:
     ```
     gh issue comment <number> --body "🔀 Reassigned from <old-subsystem> to <new-subsystem> by session <session-id>.

     Diagnosis: [root cause summary]
     Evidence: [file:line, command output]
     The defect is in [component] because [reason]."
     ```
  3. If you can fix the other component, keep the claim and fix it.
     Otherwise release the claim; the diagnosis comment stays.
- **Stale claims** (a claim with no follow-up comments or referencing
  commits): never override another agent's claim. Only the user decides
  whether to release it.

## Triage: Component Assignment and Prioritization

### Component verification pass

Before fixing anything, check every open issue's labels:

```
gh issue list --state open --json number,title,labels
```

- Zero subsystem labels: read the issue and assign one. Several: keep the one
  where the defect lives, not where the symptom appears, and remove the rest.
- Check the label against the title and body; a quick read is enough. A
  label you can tell is wrong from the title alone (e.g. a code-generator
  panic labeled with the parser's subsystem): relabel it now.
- Read titles, bodies, and labels, not code; full diagnosis happens when
  fixing.
- Comment your reasoning on every relabeled issue.

### Picking issues by component

Claim only issues labeled for the component you are working on:

```
gh issue list --state open --label "<subsystem>" --json number,title,labels
```

If diagnosis puts the root cause elsewhere, reassign (see Claiming).

### Prioritizing within a component

1. blocker
2. bug + regression
3. bug
4. enhancement

Within one priority, prefer issues with more evidence (exact reproduction,
file:line).

### Fixing

Claim and fix one issue at a time with `fix-test` discipline; do not combine
issues. After each fix, check whether other issues in the component are now
resolved, and close those with evidence.

## Integration

- **orchestrate**: an issue can be a milestone: the orchestrator briefs it,
  the implementation agent fixes it, the audit verifies it, and it is closed
  on commit.
- **project-creation**: link issues to PROJECT.md acceptance criteria or
  PLAN.md milestones.
- **regressions**: unexpected regressions become issues; declared regressions
  reference their resolving milestone, not an issue.

## References

- `references/issue-template.md` — issue body structure
- `references/bug-report-template.md` — repo-local bug report structure
- `references/rationale.md` — why claims and labels work this way
- `../references/project-profile.md` — profile §Repository holds the tracker
  and labels
