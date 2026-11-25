# Decision Protocol

Read the latest AUDIT.md section and route.

Attribute every failure before routing it, per
`../../references/failure-attribution.md`.

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

At most one fix round per step. If the re-audit fails, or the fix round
itself introduced a blocker, the step returns to design and the user is
told; it does not get a second fix round.

The escalation names the attribution: the violated contract, the component
that owns it, the milestone that owns the work, and which suite or fixture
decided that. If no fixture covers the boundary the failure crossed, say so —
the missing fixture is part of the remediation, because without it the next
failure on that seam is attributed by argument again.

A recommended remediation that repairs a lower layer's defect in the current
layer is not a remediation. Say plainly that the fix belongs elsewhere.

## Structural Issues (functional pass)

Functional criteria pass but structural issues exist. Present findings
to user — they decide whether to fix now or accept.

## Incomplete (progress without completion)

The worker returned findings or diagnosis instead of a patch + verification
or a concrete blocker. This is not a valid exit.

**Special case — returned mid-verification.** If the patch is complete in the
working tree and the last `[checkpoint]` says a long verification was started
or pending, this is a premature exit, not a scope problem: recover it per
`../SKILL.md`, "Long-Running Verification", rule 2.

Otherwise, do NOT re-run the same brief. Instead:

1. Read STATUS.md for what was accomplished vs remaining.
2. Keep the remaining work in the current milestone. Sequential failures are
   not new phases.
3. Propose a plan split only per `brief-template.md`, "Scope validation" and
   "Size by risk and reward", and obtain approval before creating it.
4. Never audit a verified subset as satisfying the original criteria.
5. Update PLAN.md only after an approved split or a real scope change.

## Between Milestones

After commit, before next milestone:

### 1. Assigned validation gate

Run the owning suite or broader gate assigned to this milestone in PLAN.md and
compare against the baseline in STATUS.md. Choose tiers per
`../../regressions/SKILL.md`, "When to Run What". Classify every change:

- **Unexpected regression** — passing at baseline, now failing, not
  declared in PLAN.md. Escalate. Attribute it before proposing any fix: the
  current milestone may have broken it or only exposed a lower layer's
  defect. Both escalate; they do not share a fix site (failure-attribution,
  "When Another Component Owns It").
- **Declared regression** — all four conditions: named in PLAN.md before
  implementation, named a resolving milestone, milestone still exists,
  recorded in STATUS.md. If any condition fails → unexpected.
- **Improvement** — was failing, now passing. Record in STATUS.md.

No retroactive declarations, vague declarations, open-ended resolution,
or baseline updates before classification. Ambiguity defaults to
unexpected — escalate.

Never resolve a classification by weakening the failing test, by compensating
for the failure in the current milestone's code, or by re-running until it
passes. Each of those converts a locatable defect into an unlocatable one.

### 2. Plan coherence

Compare AUDIT.md against PLAN.md. Update PLAN.md to match reality. If a
resolving milestone for a declared regression was removed, those
regressions become unexpected — escalate. No retroactive declarations.

### 3. Cumulative progress

Re-read PROJECT.md criteria. Verify remaining milestones cover all
criteria. Add milestones or escalate if gaps exist. Update STATUS.md.
