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

- command: [the regression command, e.g. scripts/full-regression.sh check]
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
