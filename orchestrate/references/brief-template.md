# BRIEF.md Template

The briefing agent writes `project-notes/<slug>/BRIEF.md` using this structure.

```md
# Task Brief: [milestone name]

## Objective

[One sentence: what this milestone achieves and why it matters.]

## Acceptance Criteria

[Numbered list. Filtered from PROJECT.md to this milestone only.
Each criterion must be verifiable — an exact command, expected output,
or observable behavior.]

1. ...
2. ...

## Relevant Files

[Every file the implementation agent will need to read or modify.
Include the file's role so the agent knows why it matters.]

- `path/to/file` — [role: defines X / implements Y / tests Z]

## Implementation Approach

[Key steps distilled from PLAN.md. Enough to guide the implementation
agent, not so much that it becomes a second plan.]

1. ...
2. ...

## Constraints

[Hard boundaries the implementation must respect.]

- Design decisions from PROJECT.md that apply here
- Forbidden approaches (e.g., no fallback inference, no silent degradation)
- Dependencies on other milestones (what must exist already)
- Files or APIs that must NOT be modified

## Done When

[Concrete verification. What the audit agent will check.]

- [ ] [command to run] produces [expected output]
- [ ] [file] contains [expected content]
- [ ] [behavior] is observable via [method]

## Declared Regressions

[Tests or behaviors that this milestone is expected to break, with the
specific milestone that will resolve each one. These must be declared
here BEFORE the implementation agent runs — regressions discovered
after implementation cannot be retroactively declared as expected.

If this milestone should not introduce any regressions, write "None."

Each declaration must be specific enough to match against test results.]

- [ ] `[test name or command surface]` will regress because [reason].
  Resolved by: [milestone name/number].

## Open Questions

[Ambiguities found during briefing. The orchestrator resolves these
before launching the implementation agent. If none, write "None."]

- ...
```
