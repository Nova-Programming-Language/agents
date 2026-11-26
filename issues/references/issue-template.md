# Issue Body Template

Use this structure when creating issues in the project's tracker
(profile §Repository).

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

- Subsystem: [the component, as a label or as the profile names it (§Repository)]
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
  not a reproduction — a single command that runs one named test or input
  file (profile §Tests) is.
- One defect per issue. If you found two bugs, file two issues; never "fix
  these 5 things".
- A good issue has: a title that states the symptom in one line, specific
  enough to find later; an exact reproduction command; expected versus actual
  behavior; evidence (file:line, error output, test names); and an ownership
  hypothesis naming the subsystem that likely owns it.
- Not a good issue: a vague title ("something is broken"), no reproduction
  steps, or speculation without evidence.
