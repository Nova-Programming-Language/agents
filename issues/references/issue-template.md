# Issue Body Template

Use this structure when creating issues with `gh issue create`.

```md
## Summary

[One sentence: what is broken and where.]

## Reproduction

```bash
[exact command to reproduce]
```

- Branch/commit: [where this was observed]
- Clean rebuild: [yes/no/not applicable]

## Expected

[What should happen.]

## Actual

[What happens instead. Include error output.]

## Evidence

- [file:line — what is wrong at this location]
- [command output or test name]

## Ownership Hypothesis

- Subsystem: [frontend | interpreter | c-backend | tests | cli | runtime | docs]
- Likely cause: [one sentence — what is probably wrong]

## Context

- Found via: [audit | regression | manual testing | code review]
- Blocks: [milestone, issue, or "nothing"]
- Related: [other issue numbers, AUDIT.md dates, or "none"]
```

## Notes

- Every field should be filled. If genuinely unknown, write "unknown"
  with what you tried — not just blank.
- The reproduction must be exact and runnable. "Do X then Y then Z" is
  not a reproduction — `nova check tests/build/foo.nova` is.
- One defect per issue. If you found two bugs, file two issues.
