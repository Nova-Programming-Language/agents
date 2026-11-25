# STATUS.md Template

```md
# Current Status

## Current Stage

- requirements | architecture | implementation_plan | implementation | validation

## Current Focus

- what this session is trying to accomplish:

## Done

- completed item:

## In Progress

- active item:

## Blocked

- blocker (must be verifiable, not a judgment call):

## Regression Baseline

Captured after each milestone commit. Used by between-milestones
regression checks.

- command: [the full-regression command from profile §Regression]
- captured at commit: [hash]
- passing: [count]
- failing: [count]
- not compilable: [count]

## Declared Regressions

Tests expected to fail due to in-progress migration. Each must name the
introducing milestone and the resolving milestone.

- [test or surface]: introduced by [M#], resolved by [M#]

## Next Session Start Here

1. exact command or file to open:
2. exact question to answer next:
3. exact proof or artifact update expected:

## Next Validation

- next command(s):
- missing-data or negative checks:

## Working State

- branch:
- last verified commit:
```

## Keeping STATUS.md current

STATUS.md answers "where is this project right now" for someone who has read
nothing else. It follows the template above and stays roughly one screen.

- **Never append to STATUS.md.** Rewrite the sections that changed and delete
  what is no longer true.
- LOG.md is append-only, and it is the only place history accumulates. It
  records software engineering work only, not agent workflow.
- `[checkpoint]` markers are working state while their milestone is open.
  When the milestone closes, move them to LOG.md and delete them from
  STATUS.md.
- Treat any of these as a defect to fix before doing anything else:
  - STATUS.md carries more than one dated section, or any superseded
    checkpoint
  - STATUS.md exceeds roughly 150 lines
  - a claim near the top is contradicted by a section further down
  - history appears in STATUS.md that is not also in LOG.md
- To migrate: cut every superseded section, append it verbatim under a dated
  LOG.md entry saying what moved and why, then rewrite STATUS.md from the
  template against the actual current state. Never drop the text; only change
  its home.

Why: the failure is silent and compounding. Each session appends "just one
dated section", STATUS grows into a second undated log, and the current state
becomes something a reader has to reconstruct by diffing checkpoints. Once
that has happened, stale claims survive at the top of the file because nobody
rewrites a 1,000-line status, and the next session reads STATUS first.
