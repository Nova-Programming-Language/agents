---
name: fix-test
description: Fix failing tests one at a time with root-cause analysis and verified fixes; use when presented with test failures to fix, regression results to work through, or a list of bugs that need depth-first resolution.
---

# Fix Test

## When to Use

Use when you have failing tests to fix. This skill enforces one-at-a-time
resolution — each failure is fully diagnosed, fixed, and verified before
touching the next.

Do NOT batch failures. Do NOT start a second fix before the first is
verified.

## The Loop

Repeat for each failure:

### 1. Select one failure

Pick the next failure from the list. If working through regression
results, use the structured failure data (classification, owner, phase)
to choose the most informative failure first — one whose fix is likely
to resolve or explain other failures.

Record in STATUS.md: which failure you are working on and why.

### 2. Reproduce

Run the failing test in isolation. Record:
- The exact command
- The actual output (error message, wrong value, crash)
- The expected output

If the test cannot be reproduced in isolation, note that — it may be
an ordering or environment dependency.

### 3. Diagnose root cause

Trace from the symptom to the root cause. This is the critical step
that must not be skipped.

- Read the test to understand what it asserts
- Read the code path the test exercises
- Identify WHERE the behavior diverges from the expectation
- Identify WHY — is it a logic error, a missing case, wrong data,
  stale contract, upstream bug?
- If the root cause is in another component, record it as a blocker
  with evidence — do not work around it

The diagnosis must name:
- The root cause (one sentence)
- The file and line where the defect lives
- Whether this is a local fix or requires upstream changes

### 4. Fix the root cause

Implement the fix. Rules:

- Fix the root cause, not the symptom. If the test fails because a
  value is nil, don't add a nil check — find out why it's nil.
- Do not modify the test expectations to match wrong behavior.
- Do not add workarounds, fallbacks, or defensive checks that hide
  the real problem.
- Keep the fix minimal — only change what is necessary.
- If the fix touches shared code, consider whether it affects other
  tests (check in step 5).

### 5. Verify

Run the target test. It must pass.

Then run a broader scope to check for collateral damage:
- If the fix touched shared code, run related tests
- If a regression suite command exists, run it and compare

Record in STATUS.md:
- Target test: pass/fail
- Broader check: any new failures introduced?

If the fix introduced new failures, you are not done. Diagnose and
fix those before moving on — they are part of THIS fix, not a
separate item.

### 6. Record and move on

Only after the fix is verified with no collateral damage:

- Update STATUS.md: move this failure to Done with a one-line summary
  of the root cause and fix
- If working in an orchestrate pipeline, the orchestrator may commit
  at this point

Then return to step 1 for the next failure.

## When to Stop

- All failures in the list are fixed and verified
- You hit a blocker (upstream bug, missing contract) — record it with
  evidence and stop. Do not work around it.
- The user tells you to stop

Do NOT stop because "the remaining failures are similar" or "these
can be batched." Each failure gets its own diagnosis.

## Depth-First Rules

These rules exist because agents consistently prefer breadth (touching
many failures shallowly) over depth (fixing each one completely):

- **One at a time.** Never have two failures in progress simultaneously.
- **Verify before moving on.** A fix is not done until the test passes
  and no new failures are introduced.
- **Root cause required.** Every fix must name the root cause. "Made
  the test pass" is not a diagnosis. If you cannot identify the root
  cause, escalate — do not guess.
- **No symptom fixes.** Adding nil checks, clamps, default values, or
  try/catch blocks that hide the real problem is not fixing. It is
  covering up.
- **No test modification.** Changing test expectations or assertions
  to match wrong output is not fixing. The test is usually right and
  the code is wrong.
- **No batching.** "These three failures have the same root cause" may
  be true, but verify it — fix one, verify, then check if the others
  are resolved. If they are, great. If not, each gets its own cycle.

## Integration with Orchestrate

When orchestrate encounters a milestone with multiple test failures:
- Each failure should be a separate cycle of this skill's loop
- The orchestrate pipeline should not pack multiple independent failures
  into one implementation agent run
- The audit agent verifies each fix individually

When used standalone, this skill manages its own sequencing.

## References

- `references/failure-log-template.md` — format for tracking fixes
