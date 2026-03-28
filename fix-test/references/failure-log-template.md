# Failure Log Template

Track each fix in STATUS.md or a dedicated log. One entry per failure.

```md
## [test name or identifier]

- status: in_progress | audit_failed | fixed | blocked
- command: [exact command to reproduce]
- symptom: [what fails — error message, wrong value, crash]
- root cause: [one sentence — the actual defect]
- defect location: [file:line]
- fix: [one sentence — what was changed]
- verified: [target test command and result]
- collateral: [broader check — any new failures? command and result]
- audit: pass | fail [if fail — what the audit found]
- commit: [hash, or "pending audit"]
- blocked by: [if blocked — upstream issue with evidence]
```

## Notes

- Every field must be filled before marking "fixed"
- "root cause" and "defect location" prove the diagnosis was done
- "audit" must be "pass" before "commit" can have a hash
- If audit fails, the status moves to "audit_failed" and the fix
  goes back to step 4 of the loop with the audit findings
- "collateral" catches fixes that solve one test but break others
- "blocked by" requires file:line or command evidence — not a
  judgment call
