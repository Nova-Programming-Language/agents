---
name: fix-test
description: Fix failing tests one at a time with root-cause analysis, audit, and verified commits; use when presented with test failures to fix, regression results to work through, or a list of bugs that need depth-first resolution.
---

# Fix Test

## When to Use

Use when you have failing tests to fix. This skill enforces one-at-a-time
resolution — each failure is diagnosed, fixed, audited, and committed
before touching the next.

Do NOT batch failures. Do NOT start a second fix before the first is
committed.

## The Loop

Repeat for each failure:

### 1. Select one failure

Pick the next failure. If working through regression results, run
`scripts/full-regression.sh failed --json` and use the resulting
records — specifically `classification`, `owner`, and `phase` — to
choose the most informative failure first — one whose fix is likely
to resolve or explain others.

Record in STATUS.md: which failure you are working on and why.

### 2. Reproduce

Run the failing test in isolation. Record:
- The exact command
- The actual output (error message, wrong value, crash)
- The expected output

If not reproducible in isolation, note it — may be an ordering or
environment dependency.

### 3. Diagnose root cause

Trace from symptom to root cause. This step must not be skipped.

- Read the relevant architecture document in `docs/architecture/` to
  identify which component owns the behavior being tested. Use the
  component's **Invariant** and **Not responsible for** sections to
  determine whether the defect is in the component itself or in an
  upstream producer.
- Read the test to understand what it asserts
- Read the code path the test exercises
- Identify WHERE behavior diverges from the expectation
- Identify WHY — logic error, missing case, wrong data, stale
  contract, upstream bug?
- If root cause is in another component, record as blocker with
  evidence — do not work around it
- If the architecture document has a "Common Change Patterns" section,
  check whether the original change that introduced the bug followed
  the pattern. Often a test failure means one component was updated
  but a co-required component was not.

The diagnosis must name:
- The root cause (one sentence)
- The file and line where the defect lives
- Whether this is a local fix or requires upstream changes

### 3a. File discovered bugs

During diagnosis you will often find bugs that are unrelated to the
current failure — upstream contract violations, adjacent defects,
stale assumptions in other components. Do not ignore them and do not
fix them inline (that is scope creep).

For each discovered bug:
- File a GitHub issue using the `issues` skill
- Include the evidence you already have (file:line, what's wrong)
- Label it with the correct subsystem
- Reference the current issue if related

Then continue with the current fix. The discovered bugs are now
tracked and will get their own fix cycle.

This is not optional. Unfiled bugs are lost bugs.

### 4. Fix the root cause

Implement the fix:

- Fix the root cause, not the symptom
- Do not modify test expectations to match wrong behavior
- Do not add workarounds or defensive checks that hide the problem
- Keep the fix minimal — do not fix discovered bugs inline, they
  have their own issues now

### 5. Verify

Run the target test. It must pass.

Then verify the component broadly. The goal is not just proving the
target test passes — it is proving the component still works across
its major code paths. A fix that passes one test but breaks the
component elsewhere is not a fix.

- Identify which component owns the code you changed, using the
  architecture document if one exists.
- Run the component's test suite or a representative subset that
  exercises its major code paths. For example, if you fixed a bug
  in the type checker, run the type checking tests broadly, not
  just the one test that was failing.
- If the fix touched shared code (utilities, data structures, common
  paths), widen further — run tests for the components that consume
  that shared code.
- If a regression suite command exists, run it and compare.

If the fix introduced new failures, diagnose and fix those before
proceeding — they are part of THIS fix.

### 6. Audit

Run the `audit` skill against this fix. The audit checks:

- **Functional**: does the target test now pass?
- **Structural**: is this a root-cause fix or a symptom fix? Does it
  introduce shortcuts, silent failures, or incomplete error handling?
- **Completeness**: does the fix match the diagnosed root cause, or
  did it drift into a different approach?
- **Collateral**: any new failures or out-of-scope changes?

If the audit fails, go back to step 4 with the audit findings. The
fix must pass audit before it can be committed. Do not skip the audit
because "it's a small change" — small changes are where shortcuts hide.

### 7. Commit

After passing audit, commit using `checkin`. The commit message must
include:
- The test that was failing
- The root cause (one sentence)
- The fix (one sentence)
- The verification command
- Issue reference: `Fixes #N` if this fully resolves the issue, or
  `Progress on #N` if the issue covers multiple failures and this
  commit addresses one of them

This creates a clean, revertible checkpoint per fix. If a later fix
goes wrong, earlier fixes are safely committed.

### 8. Move on

Update STATUS.md: move this failure to Done with the commit hash.

Check if other failures in the list are now resolved by this fix —
run them, and if they pass, mark them Done referencing this commit.
Do not assume they're fixed without running them.

Return to step 1 for the next failure.

## When to Stop

- All failures are fixed, audited, and committed
- You hit a blocker (upstream bug, missing contract) — record with
  evidence and stop. Do not work around it.
- The user tells you to stop

Do NOT stop because "the remaining failures are similar" or "these
can be batched."

## Depth-First Rules

- **One at a time.** Never have two failures in progress simultaneously.
- **Audit before commit.** Every fix is audited. No exceptions.
- **Commit before moving on.** A fix is not done until it is committed.
- **Root cause required.** "Made the test pass" is not a diagnosis.
- **No symptom fixes.** Nil checks, clamps, defaults, try/catch that
  hide the problem are not fixes.
- **No test modification.** The test is usually right and the code
  is wrong.
- **No batching.** Fix one, verify, audit, commit. Then check if
  others resolved.

## Integration with Orchestrate

When orchestrate encounters test failures:
- Each failure is a separate cycle of this loop
- The orchestrate pipeline should not batch failures into one agent run
- Each cycle produces its own audit and commit

When used standalone, this skill manages its own sequencing.

## References

- `references/failure-log-template.md` — format for tracking fixes
- `audit/SKILL.md` — audit skill used at step 6
