---
name: fix-test
description: Fix failing tests one at a time, each with root-cause diagnosis, audit, and commit; use for test failures or regression results to work through. Not for new features (use coding).
---

# Fix Test

## When to Use

Use when you have failing tests to fix. This skill enforces one-at-a-time
resolution — each failure is diagnosed, fixed, audited, and committed
before touching the next.

Do NOT batch failures. Do NOT start a second fix before the first is
committed.

## The Loop

Repeat for each failure. Track each one per
`references/failure-log-template.md` (in STATUS.md or a dedicated log).

### 1. Select one failure

- Pick the next failure. From regression results, use the structured failure
  records (profile §Regression): their classification, owner, and phase pick
  the most informative failure first, one whose fix is likely to resolve or
  explain others.
- Treat the record's owner as a starting hypothesis, not a verdict; step 3
  decides the owning component.
- Record in STATUS.md which failure you are working on and why.

### 2. Reproduce

- For a package-repository failure, first complete
  `../references/package-onboarding.md`, reading the root and target package
  READMEs before choosing the reproduction command.
- Run the failing test in isolation with the single-test command (profile
  §Tests), keeping the exact scope; profile §Tests says which replay options
  reproduce an earlier scope. Record the exact command, the actual output, and
  the expected output.
- Confirm the executed count per `../regressions/SKILL.md`, "Non-Negotiable
  Rules", before concluding a failure does not reproduce.
- If it does not reproduce in isolation, note it: it may be an ordering or
  environment dependency.

### 3. Diagnose root cause

Never skip this step. Trace from symptom to root cause:

- Find the owning component and its co-required changes per
  `../coding/SKILL.md`, "Using Architecture Documents". Check whether the
  change that introduced the bug followed the matching Common Change Pattern.
- Read the test and the code path it exercises; find where behavior diverges
  from the expectation, and why.
- Attribute the failure per `../references/failure-attribution.md`. If the
  root cause is in another component, record it as a blocker with evidence;
  do not work around it.

The diagnosis must name:

- the root cause (one sentence)
- the rule it violates, and every site that enforces or depends on that rule
  (file:line), including other implementations of the same check
- the file and line where the defect lives
- the component that owns the violated contract, and where that contract is
  written (the fix site)
- whether the fix is local or needs upstream changes
- which suite or fixture decided the attribution, and whether a boundary the
  failure crossed lacks a test on either side

### 3a. File discovered bugs

- File every unrelated bug found during diagnosis with the `issues` skill:
  evidence (file:line, what is wrong), the subsystem label, and a reference to
  the current issue if related, then continue with the current fix. This is
  not optional: unfiled bugs are lost bugs.
- Do not fix them inline, and do not ignore them.
- Sites in the same class as the current defect are not discovered bugs; they
  are part of this fix.

### 4. Fix the root cause

- Fix the whole class per `../coding/SKILL.md`, "Component Workflow", step 1.
- Do not modify test expectations to match wrong behavior, and do not add
  workarounds or defensive checks that hide the problem.

### 5. Verify

- Run the target test; it must pass.
- Add tests across the class's sites, not only the original reproduction.
- Verify the component broadly: run the tests that own the changed code — the
  owning component's suite or a representative subset that exercises its
  major code paths — and, when the fix touched shared code, the tests of the
  components that consume it (`../regressions/SKILL.md`, "When to Run What").
  Exhaustive tests wait for the end of the milestone; used standalone, run
  them once after the last fix.
- Land the regression test and re-run the exposing path per
  `../references/failure-attribution.md`, "Validation Expectations". If the
  symptom survives a fix at the owning component, the attribution was wrong
  and step 3 is not finished.
- Any new failure the fix introduced is part of this fix: diagnose and fix it
  before going on.

### 6. Audit

- Run the `audit` skill on this fix (`../audit/SKILL.md`), even for a small
  change. Besides its own dimensions, it checks that the fix matches the
  diagnosed root cause rather than drifting into a different approach, and
  for collateral: new failures or out-of-scope changes.
- If the audit fails, go back to step 4 with its findings. Do not commit until
  it passes.

### 7. Commit

Commit with `checkin`. The message names:

- the test that was failing
- the root cause (one sentence)
- the fix (one sentence)
- the key verification command, in one line (`checkin`, Commit Policy)
- the issue reference, in the closing or progress form `checkin` describes

### 8. Move on

- Update STATUS.md: move this failure to Done with the commit hash.
- Run the other failures in the list; mark any that now pass Done, referencing
  this commit. Do not assume they are fixed without running them.
- Return to step 1.

## When to Stop

- All failures are fixed, audited, and committed.
- You hit a blocker (upstream bug, missing contract): record it with evidence
  and stop. Do not work around it.
- The user tells you to stop.

Never stop because "the remaining failures are similar" or "these can be
batched".

## Depth-First Rules

- **One at a time.** Never have two failures in progress at once.
- **Audit before commit.** Every fix is audited. No exceptions.
- **Commit before moving on.** A fix is not done until it is committed.
- **Root cause required.** "Made the test pass" is not a diagnosis.
- **A flaky test is a bug.** Handle it per `../regressions/SKILL.md`,
  "Non-Negotiable Rules"; never rerun-until-green and move on.
- **Fix at the owning component**, per `../references/failure-attribution.md`:
  not at the frame where the value was observed, not in the consumer that
  received it, and not in the package that happens to be in scope.
- **No symptom fixes.** Nil checks, clamps, defaults, and try/catch that hide
  the problem are not fixes.
- **No test modification.** The test is usually right and the code is wrong.

## Integration with Orchestrate

Each failure is its own cycle of this loop, with its own audit and commit; the
orchestrate pipeline does not combine failures into one agent run. Used
standalone, this skill manages its own sequencing.

## References

- `../references/project-profile.md` — where project commands live (cited as
  "profile §Section")
- `../references/package-onboarding.md` — required before package failure work
- `../references/failure-attribution.md` — which component owns the defect
- `../coding/SKILL.md` — fix the class; architecture docs
- `../regressions/SKILL.md` — which tests to run; flaky tests; empty scopes
- `references/failure-log-template.md` — format for tracking fixes
- `references/rationale.md` — why the loop is shaped this way
- `../audit/SKILL.md` — audit skill used at step 6
