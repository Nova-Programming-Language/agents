# Failure Attribution

Use this reference when something fails and it is not yet established which
component owns the defect or which milestone owns the work.

## Core Rule

A failure raises two questions, answered separately and in this order:

1. **Structural** — whose contract was violated? This decides **where the fix
   goes**.
2. **Temporal** — which milestone introduced it, or newly exposed it? This
   decides **who does the work now and how it is tracked**.

The component where a failure surfaces is usually not the component that owns
it. Attribution is decided by which suite or fixture fails, never by which file
is convenient to change. A fix applied anywhere but the owning component is an
abstraction violation even when it turns the test green — it leaves the real
defect in place and adds a second one.

## The Buckets

Apply in order; the first match wins.

1. **A lower layer's own suite fails** → that layer owns the defect. It passed
   at that layer's recorded exit commit, so bisect names the commit that broke
   it. If the breaking commit belongs to the current milestone, the current
   milestone owns the *work* and the lower layer still owns the *fix site*.
2. **Lower suites pass, the producer-side fixture of the boundary under
   construction fails** → the current milestone. The layer emitted something
   its own contract does not describe.
3. **The producer fixture passes, the consumer-side fixture fails** → the
   downstream component. It was handed a conforming artifact and mishandled
   it. Do not repair this by changing the conforming output.
4. **Every fixture passes and the integrated path still fails** → the current
   milestone owns the work, and the missing fixture is itself a finding.
   "Nothing covers this boundary" is a gap to close, never a licence to
   attribute by opinion.

## Required Questions

1. What contract does the observed behavior violate, and where is that
   contract written?
2. Which component owns producing the value that is wrong?
3. Which suite should have caught this, and did it run?
4. At which recorded commit did the owning layer's suite last pass?
5. If the fix is proposed outside the owning component, what invariant does
   that placement break?

If question 5 has an answer, the fix is in the wrong place.

## Boundary Fixtures

Every seam between components needs a producer-side and a consumer-side test
against the **same recorded artifact** — the emitted manifest, the wire frame,
the patch list, the request object. Without both sides, bucket 3 is
undecidable and every cross-component failure degrades into argument. Adding
the missing fixture is part of the fix, not follow-up work.

A second implementation of a boundary — a test backend, a fake producer — is
worth its cost wherever it separates "our logic is wrong" from "the other side
is wrong" without reasoning.

## Forbidden Patterns

- Compensating downstream for an upstream defect: shaping consumer code around
  a producer bug, or adding a package-side workaround for a toolchain bug.
  Fix the producer.
- Weakening, skipping, or deleting a lower layer's test so the current
  milestone goes green.
- Moving a check to where the failure surfaced rather than where the contract
  lives.
- Changing a producer's conforming output to absorb a consumer's mishandling.
- Relaxing acceptance criteria, or re-scoping the milestone, in place of
  attributing the failure.
- Re-running until green, or recording a failure as transient or flaky. Every
  failure is root-caused.
- A "temporary" shim at the wrong layer with no issue and no resolving
  milestone.
- Attributing by opinion when no fixture covers the boundary, instead of
  adding the fixture.
- Silently widening the milestone to repair another layer. The repair may be
  correct; it is tracked, not absorbed.

## When Another Component Owns It

- File it, with the reproduction and the violated contract named.
- If it blocks the current milestone's acceptance criteria, stop with a
  blocker naming that issue. Do not work around it to reach green.
- If it does not block, record it and continue; it is not carried silently and
  it is not fixed opportunistically inside an unrelated milestone.
- A defect in a prior milestone that a later one exposes is still that
  milestone's defect. Exposure is not authorship, and the exposing milestone
  does not inherit the fix site.

## Validation Expectations

- The fix lands with a test **in the owning component** that fails before and
  passes after.
- When the failure crossed a boundary, both sides' fixtures exist afterwards.
- A newly exposed prior defect gets its regression test at the layer that owns
  it, not only at the layer that surfaced it.
- Re-run the exposing path as well, to confirm the symptom is gone and the
  attribution was right.

## Reporting Expectation

State:

- the observed failure
- the contract violated, and where it is written
- the owning component, and the milestone that owns the work
- which suite or fixture decided the attribution
- whether a fixture had to be added to make the attribution decidable
