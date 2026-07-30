# Decision Protocol

Read the latest AUDIT.md section and route.

## Pass

All dimensions clean.

1. Commit using `checkin`. Record commit hash in STATUS.md.
2. Mark milestone complete, set next milestone in STATUS.md.
3. Append milestone summary to LOG.md (software work only).
4. Run between-milestones checks before starting next milestone.

An audit cannot pass only a subset of the original acceptance criteria. A
public-contract milestone passes only when every promised branch and
applicable completion-matrix cell is proven.

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

**Special case — returned mid-verification.** If the patch is complete
in the working tree and the last `[checkpoint]` says a long
verification was started or pending, this is a premature exit, not a
scope problem. Do not re-brief and do not spawn a continuation agent
(it cannot inherit the worker's context and will hit the same wall;
any background run the worker started died with it). The orchestrator
re-runs the verification as a main-session background task, commits per
the brief on success, records the outcome in STATUS.md, and proceeds to
audit. See SKILL.md §Long-Running Verification.

Otherwise, do NOT re-run the same brief. Instead:

1. Read STATUS.md for what was accomplished vs remaining.
2. Keep the remaining work in the current milestone. Sequential failures are
   not new phases.
3. Propose a plan split only if evidence establishes independent root causes
   or independently auditable/releasable outcomes; obtain approval before
   creating `a`/`b`, pilot, cleanup, or follow-up milestones.
4. Never audit a verified subset as satisfying the original criteria.
5. Update PLAN.md only after an approved split or a real scope change.

## Between Milestones

After commit, before next milestone:

### 1. Assigned validation gate

Run the owning suite or broader gate assigned to this milestone in PLAN.md and
compare against the baseline in STATUS.md. Run canonical/full regression only
at defined production-changing milestone gates and final release, not after
every file, checkpoint, or documentation-only milestone.
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
