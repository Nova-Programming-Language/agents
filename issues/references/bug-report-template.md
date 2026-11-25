# Bug Report Template

Structure for a repo-local bug report (`../SKILL.md`, "Local Bug Report").
Every section is required.

## Summary

One short paragraph describing the defect and the affected workflow.

## Reproduction

```bash
# exact commands
```

Input files:

- list the specific files involved

## Expected

Describe the correct behavior.

## Actual

Describe the observed behavior and include the key error text or symptom.

## Scope

- affected commands
- which project component it is visible in
- whether the problem blocks other workflows

## Ownership Hypothesis

State the most likely subsystem responsible and why.

## Verification Notes

- branch or commit tested
- whether the worktree was clean or dirty
- whether a clean rebuild was performed
- whether the bug still reproduces on the current default branch

## Next Action

State the next concrete engineering step needed to fix or further isolate the
bug.
