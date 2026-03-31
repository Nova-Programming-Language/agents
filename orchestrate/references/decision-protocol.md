# Decision Protocol

Read the latest AUDIT.md section and route.

## Pass

All dimensions clean.

1. Commit using `checkin`. Record commit hash in STATUS.md.
2. Mark milestone complete, set next milestone in STATUS.md.
3. Append milestone summary to LOG.md (software work only).
4. Run between-milestones checks before starting next milestone.

## Fail

Any dimension has issues. Escalate to the user with AUDIT.md findings,
failed dimension(s), and recommended remediation. Wait for user direction.
No automatic retries.

## Structural Issues (functional pass)

Functional criteria pass but structural issues exist. Present findings
to user — they decide whether to fix now or accept.

## Incomplete (progress without completion)

The worker returned findings or diagnosis instead of a patch + verification
or a concrete blocker. This is not a valid exit.

Do NOT re-run the same brief. Instead:

1. Read STATUS.md for what was accomplished vs remaining.
2. If the worker filed issues for discovered defects, those become new
   milestone candidates.
3. If a partial fix was committed and verified for a subset of criteria,
   audit what was completed, then create new milestones for the rest.
4. If no code was committed, split the original milestone by distinct
   root cause — one diagnosis + fix + verification per new milestone.
5. Update PLAN.md before continuing.

Watch for: worker clears one failure, exposes the next, reports it,
repeats. This is unbounded sequential discovery. Stop, split, re-brief.

## Between Milestones

After commit, before next milestone:

### 1. Regression check

Run the test suite and compare against the baseline in STATUS.md.
Classify every change:

- **Unexpected regression** — passing at baseline, now failing, not
  declared in PLAN.md. Escalate.
- **Declared regression** — all four conditions: named in PLAN.md before
  implementation, named a resolving milestone, milestone still exists,
  recorded in STATUS.md. If any condition fails → unexpected.
- **Improvement** — was failing, now passing. Record in STATUS.md.

No retroactive declarations, vague declarations, open-ended resolution,
or baseline updates before classification. Ambiguity defaults to
unexpected — escalate.

### 2. Plan coherence

Compare AUDIT.md against PLAN.md. Update PLAN.md to match reality. If a
resolving milestone for a declared regression was removed, those
regressions become unexpected — escalate. No retroactive declarations.

### 3. Cumulative progress

Re-read PROJECT.md criteria. Verify remaining milestones cover all
criteria. Add milestones or escalate if gaps exist. Update STATUS.md.
