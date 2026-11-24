# Delivery Checks

Use this reference for project framing, planning, and critique. The
project-specific items (contracts, surfaces, and risk areas particular to the
project) are in profile §Delivery checks; find the profile per
`project-profile.md`.

## Core Questions

Every non-trivial project should answer:

1. Which command or execution surfaces are affected (profile §Command
   surfaces)?
2. Which specs or architecture docs are authoritative?
3. Does the work touch any project contract or boundary listed in profile
   §Delivery checks?
4. What user-visible behavior changes, and what must stay unchanged?
5. What must fail loudly if required data or state is missing?
6. Which related constructs must share a path, and where do they truly diverge?
7. What exact commands prove the work is done?

## Affected Surfaces

Check the ones in profile §Command surfaces that actually apply, including
execution engines or backends and docs/examples.

Do not leave this implicit for multi-session projects.

## Authoritative Sources

Name the relevant sources explicitly:

- spec docs and architecture docs (profile §Paths)
- the contract documents and runtime or ABI assumptions listed in profile
  §Delivery checks

## Risk Areas

Call these out when relevant, along with the project-specific risks in
profile §Delivery checks:

- fallback inference instead of contract data
- cross-command inconsistency
- parity drift between execution engines or backends
- stale build or runtime artifacts
- syntax-shaped special casing
- silent missing-data handling instead of loud failure
- drift in a cross-component data contract

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

- how many tests ran (a run that selected nothing may exit zero; profile
  §Tests says how the runner reports the executed count)
- how many failed in total, not only how many are new
- for a comparison-based suite, both numbers: "no new regressions" describes a
  delta against a baseline and is compatible with a non-zero failure count

A suite that reports no new regressions has not reported a passing suite.
