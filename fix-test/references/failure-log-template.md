# Failure Log Template

Track each fix in STATUS.md or a dedicated log. One entry per failure.

```md
## [test name or identifier]

- status: in_progress | fixed | blocked
- command: [exact command to reproduce]
- symptom: [what fails — error message, wrong value, crash]
- root cause: [one sentence — the actual defect]
- defect location: [file:line]
- fix: [one sentence — what was changed]
- verified: [command and result — target test passes]
- collateral: [broader check — any new failures? command and result]
- blocked by: [if blocked — what upstream issue, with evidence]
```

## Notes

- Every field must be filled before marking "fixed"
- "root cause" and "defect location" are required — they prove the
  diagnosis was done, not just a guess-and-check
- "collateral" catches fixes that solve one test but break others
- If "blocked by" is filled, include file:line or command evidence —
  not a judgment call like "this seems hard"
