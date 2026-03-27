# Decision Protocol

After each audit, the orchestrator reads the latest dated section in
`project-notes/<slug>/AUDIT.md` and routes based on three dimensions:
functional, structural, and forward impact.

## Routing Matrix

| Functional | Structural | Forward | Route |
|---|---|---|---|
| pass | clean | clean | **proceed** |
| pass | issues | any | **fix_structure** (then re-audit) |
| pass | clean | risks | **adjust_plan** then proceed |
| fail | any | any | **retry** or **escalate** |
| partial | any | any | evaluate per Partial rules below |

A milestone is not complete until all three dimensions are clean.
Structural issues are never deferred as tech debt — they are fixed or
escalated before the orchestrator moves on.

## Proceed

All functional criteria pass. No structural issues. No forward risks.

1. Commit the milestone using the `checkin` skill. The commit message
   should name the milestone and summarize what changed. This creates
   a clean base commit for the next milestone's audit diff.
2. Update STATUS.md: mark milestone complete, set next milestone as focus.
   Record the commit hash as the last verified commit.
3. Append milestone summary to LOG.md.
4. BRIEF.md is overwritten on the next milestone. AUDIT.md is append-only
   — prior sections remain as history.
5. Start the next milestone's pipeline.

## Fix Structure

Functional criteria pass, but the audit found structural issues
(wrong layer, missing abstraction, semantic duplication, pattern breaks).

Every structural finding must be resolved before the milestone is complete:

- **Fix**: Spawn a targeted implementation agent whose brief contains only
  the structural fixes from AUDIT.md. Re-audit after. The milestone
  does not pass until the re-audit comes back clean.
- **Escalate**: Structural issues that require an architectural decision
  the orchestrator should not make alone. Ask the user. Do not proceed
  until the user has responded and the fix is applied.

Structural issues are never logged-and-skipped. The cost of fixing now
is always lower than the cost of compounding across milestones.

## Adjust Plan

Functional criteria pass, but the audit found forward risks (wrong
interface shape, tight coupling, decisions that will need undoing).

1. Read the forward impact section of the latest AUDIT.md entry.
2. If the audit suggests a "small fix now" — spawn a targeted
   implementation agent to make that fix. Re-audit after.
3. If the risk requires changing milestone scope or sequencing — update
   PLAN.md and STATUS.md before proceeding. If the change is significant,
   ask the user first.
4. If the risk is speculative (might matter, might not) — note it in
   STATUS.md and proceed. Revisit if the next milestone hits it.

## Retry (functional fail)

One or more functional criteria fail.

### Attempt < 3

1. Extract the failing criteria, evidence, and any structural findings
   from the latest AUDIT.md section.
2. Append them to the implementation agent's prompt under
   "PREVIOUS ATTEMPT FAILED" (see agent-briefs.md).
3. Include relevant structural findings — they may explain why the
   functional criteria failed (e.g., wrong layer, missed abstraction).
4. Re-launch the implementation agent (step 2 of the pipeline).
5. Re-launch the audit agent (step 3 of the pipeline).
6. Increment retry count.

### Attempt >= 3

1. Present to the user:
   - Which criteria are still failing
   - What each retry attempted
   - Structural findings that may be root causes
   - The audit agent's hypothesis (from AUDIT.md)
2. Wait for user direction before continuing.
3. If the user provides a fix or guidance, reset retry count and re-enter
   the pipeline.

## Partial

Some functional criteria pass, some cannot_verify.

- If cannot_verify is due to missing test infrastructure or environment:
  note in STATUS.md, treat passing criteria as sufficient, proceed.
  Still evaluate structural and forward dimensions.
- If cannot_verify is due to ambiguous requirements:
  escalate to user to clarify, then re-run audit.
- Otherwise treat as Fail.

## Ambiguous

The audit agent could not produce a clear verdict.

1. Check if BRIEF.md was specific enough. If not, re-run the briefing
   agent with more guidance.
2. If the problem is at the requirements level (unclear what "correct"
   means), escalate to user.
3. If the problem is technical (test harness broken, environment issue),
   attempt to fix the tooling before retrying.

## Between Milestones

After a milestone passes audit and is committed, before starting the next:

### 1. Regression check

Run the project's regression or test suite and compare against the
**baseline** — the regression state recorded in STATUS.md before this
milestone's implementation began. The baseline is not "zero failures";
it is the set of tests that were passing at the prior checkpoint.

#### Running the suite

- If the project defines proof commands in PROJECT.md (e.g.,
  `scripts/full-regression.sh check`), run them.
- If individual milestones defined proof commands in PLAN.md, re-run
  the proof commands from all completed milestones, not just the latest.

#### Classifying results

Compare against the baseline and classify every change. The
classification rules are strict — there is no discretionary category.

**Unexpected regression** — a test that was passing at baseline and is
now failing, and the regression was not declared in PLAN.md before
this milestone was implemented. This is always a blocking problem.
Spawn an implementation agent to fix it, re-audit, and re-commit
before proceeding. No exceptions.

**Declared regression** — a test that is now failing and was explicitly
declared in PLAN.md *before the milestone was implemented*. All four
conditions must be met:

1. PLAN.md named the specific tests or behavior that would regress,
   written before the implementation agent ran — not added after the
   regression was observed.
2. PLAN.md named the specific milestone (by number/name) that will
   resolve each declared regression.
3. That resolving milestone still exists in PLAN.md and has not been
   completed or removed.
4. The declared regression is recorded in STATUS.md with the milestone
   that introduced it and the milestone that will resolve it.

If any of the four conditions is not met, the regression is classified
as unexpected regardless of what PLAN.md says now.

**Improvement** — a test that was failing at baseline and is now
passing. Record in STATUS.md and LOG.md.

**No change** — a test with the same result as baseline. No action.

#### Anti-circumvention rules

These rules exist to prevent agents (or the orchestrator itself) from
rationalizing regressions away:

- **No retroactive declarations.** The plan coherence step (below) may
  update PLAN.md to reflect implementation divergence, but it MUST NOT
  add new declared regressions after the regression has been observed.
  A regression that was not declared before implementation is always
  unexpected. The only way to declare a regression is in PLAN.md
  before the implementation agent runs.
- **No vague declarations.** "Some tests may break" or "callers will
  need updating" does not count. Declarations must name specific tests,
  specific command surfaces, or specific behavior changes. If the
  declaration is not specific enough to match against actual test
  results, it does not qualify.
- **No open-ended resolution.** The resolving milestone must be a
  concrete milestone with a defined scope, not "a future milestone"
  or "cleanup." If the resolving milestone is removed or descoped
  during plan coherence, all regressions it was supposed to resolve
  become unexpected and must be fixed immediately.
- **No baseline manipulation.** The baseline is updated only after
  all regressions in the current check have been classified and
  resolved or properly declared. Never update the baseline before
  classification — that would hide regressions.
- **Ambiguity is not a loophole.** If a regression cannot be clearly
  matched to a pre-existing declaration, it is unexpected. Escalate
  to the user if genuinely unsure, but do not self-resolve the
  ambiguity in favor of "expected."

#### Updating the baseline

After all regressions are classified and unexpected regressions are
resolved:

- Record the new test results as the baseline in STATUS.md.
- For declared regressions, record them alongside the baseline with
  their introducing and resolving milestones. They are tracked
  separately from the baseline — they do not become part of the
  "expected failures" set. They are temporary exceptions with an
  explicit expiration (the resolving milestone).

This is the only place where the orchestrator runs validation commands
directly (or spawns an agent to do so). It is a cross-milestone
integration check, not a repeat of the per-milestone audit.

### 2. Plan coherence

Re-read PLAN.md. Compare what was actually implemented (from the commit
and AUDIT.md) against what was planned:

- If the implementation diverged from the plan (different interface,
  different file structure, unexpected constraint), update PLAN.md to
  reflect reality. The plan must describe the world as it is, not as
  it was expected to be.
- If the divergence affects the scope or approach of upcoming milestones,
  update those milestone descriptions too.
- If a resolving milestone for a declared regression was removed or
  descoped, those regressions immediately become unexpected. Fix them
  before proceeding.
- Plan coherence updates MUST NOT add new regression declarations.
  Regression declarations can only be added to PLAN.md before a
  milestone's implementation agent runs, not during the
  between-milestones check.
- If the divergence is significant enough to change the project's
  acceptance criteria or architecture, escalate to the user before
  continuing.

### 3. Cumulative progress check

Re-read PROJECT.md. For each project-level acceptance criterion, assess
whether cumulative progress is on track:

- Which criteria are now fully or partially satisfied?
- Which criteria are not yet addressed but are covered by remaining
  milestones?
- Are there any criteria that no remaining milestone addresses?

If a gap is found (an acceptance criterion that no milestone will
satisfy), either add a milestone to PLAN.md or escalate to the user.

Update STATUS.md with the cumulative assessment before starting the
next milestone.

### 4. Proceed or escalate

If regressions are clean, the plan is coherent, and cumulative progress
is on track — proceed to the next milestone's pipeline.

If any of the three checks raised issues, resolve them before starting
the next milestone. Do not carry unresolved cross-milestone problems
into a new brief/implement/audit cycle.
