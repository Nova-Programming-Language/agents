# Nova Delivery Checks

Use this reference for Nova-specific project framing, planning, and critique.

## Core Questions

Every non-trivial Nova project should answer:

1. Which Nova command or execution surfaces are affected?
2. Which specs or architecture docs are authoritative?
3. Does the work touch lowered metadata, `prelude.novi`, runtime ABI, or
   compiled backend behavior?
4. What user-visible behavior changes, and what must stay unchanged?
5. What must fail loudly if required data or state is missing?
6. Which related constructs must share a path, and where do they truly diverge?
7. What exact commands prove the work is done?

## Affected Nova Surfaces

Check the ones that actually apply:

- `nova run`
- `nova check`
- `nova repl`
- `nova test`
- `nova doc`
- `nova-test-runner`
- interpreter-only execution
- compiled backend execution
- docs/examples

Do not leave this implicit for multi-session projects.

## Authoritative Sources

Name the relevant sources explicitly:

- language spec docs under `docs/specs/`
- architecture docs under `docs/architecture/`
- metadata contract docs
- runtime contract in `prelude.novi`
- runtime ABI or native-library assumptions

## Nova-Specific Risk Areas

Call these out when relevant:

- fallback inference instead of contract data
- cross-command inconsistency
- interpreter / compiled parity drift
- stale release or runtime artifacts
- syntax-shaped special casing
- silent missing-data handling instead of loud failure
- metadata contract drift

## Validation Expectations

Validation should identify:

- exact commands
- cross-surface checks
- negative or missing-data checks
- build or environment prerequisites
- paired tests for related constructs when needed

“Tests under this directory should pass” is not enough by itself.

When reporting a result, say what actually happened rather than what the tool
concluded:

- how many tests ran (`tests run:` — a run that selected nothing exits zero and
  reports `tests run:       0`)
- how many failed in total, not only how many are new
- for a comparison-based suite, both numbers: "no new regressions" describes a
  delta against a baseline and is compatible with a non-zero failure count

A suite that reports no new regressions has not reported a passing suite.
