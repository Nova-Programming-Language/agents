# Decision Protocol

Read the latest AUDIT.md section and route.

## Pass

All dimensions clean: functional criteria pass, no structural issues,
no forward risks.

1. Commit using `checkin`. Record commit hash in STATUS.md.
2. Update STATUS.md: mark milestone complete, set next milestone.
3. Append milestone summary to LOG.md (software work only — what code
   changed, what was proven, the next coding step).
4. Run between-milestones checks before starting next milestone.

## Fail

Any dimension has issues.

Escalate to the user with:
- The AUDIT.md findings
- Which dimension(s) failed
- The audit's recommended remediation

Wait for user direction. The user may provide guidance, adjust the plan,
fix code themselves, or descope. Do not attempt automatic fixes.

## Incomplete (progress without completion)

The implementation agent returned findings, diagnosis notes, or partial
progress instead of a patch + verification or a concrete blocker. This
is not a valid exit state.

Do NOT re-run the same brief. The milestone is likely too broad or the
worker hit sequential discoveries. Instead:

1. Read STATUS.md to understand what was accomplished vs what remains.
2. If the worker filed GitHub issues for discovered defects, those
   issues become candidates for new milestones.
3. If the worker's partial fix is committed and verified for a subset
   of the acceptance criteria, treat it as a narrower "done" — audit
   what was actually completed, then create new milestones for the
   remaining criteria.
4. If no code was committed, the milestone produced only diagnosis.
   Split the original milestone into smaller milestones based on the
   distinct root causes the worker identified. Each new milestone
   should target one root cause with one diagnosis + fix + verification.
5. Update PLAN.md with the new milestones before continuing.

The pattern to watch for: the worker clears one failure, exposes the
next, reports it, clears that one, exposes the next. This is unbounded
sequential discovery. The correct response is to stop, split, and
re-brief — not to keep running the same worker.

## Structural Issues (functional pass)

If functional criteria pass but structural issues exist, present the
structural findings to the user. Structural issues are not automatically
deferred — the user decides whether to fix now or accept.

## Between Milestones

After commit, before starting the next milestone:

### 1. Regression check

Run the project's test suite and compare against the baseline recorded
in STATUS.md (under "Regression Baseline"). The baseline is not zero
failures — it is the state at the prior checkpoint.

Classify every change:

**Unexpected regression** — passing at baseline, now failing, not
declared in PLAN.md before implementation. Escalate to user.

**Declared regression** — all four conditions must hold:
1. PLAN.md named the specific tests/behavior before implementation ran
2. PLAN.md named the specific resolving milestone
3. That resolving milestone still exists in PLAN.md
4. Recorded in STATUS.md under "Declared Regressions"

If any condition fails, the regression is unexpected.

**Improvement** — was failing, now passing. Record in STATUS.md.

Anti-circumvention rules:
- No retroactive declarations (cannot add after observing the failure)
- No vague declarations (must name specific tests or surfaces)
- No open-ended resolution (resolving milestone must be concrete)
- No baseline updates before classification
- Ambiguity defaults to unexpected — escalate rather than self-resolve

Update baseline in STATUS.md after classification.

### 2. Plan coherence

Compare AUDIT.md's description of what was implemented against PLAN.md.
Update PLAN.md to match reality if they diverged. If a resolving
milestone for a declared regression was removed, those regressions
become unexpected — escalate.

Plan coherence MUST NOT add new regression declarations after the fact.

### 3. Cumulative progress

Re-read PROJECT.md acceptance criteria. Verify remaining milestones
cover all criteria. If a gap exists, add a milestone or escalate.

Update STATUS.md with cumulative assessment before starting the next
milestone.
