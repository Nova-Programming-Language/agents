# Project Stage Checklists

Use this file after reading `PROJECT.md`, `PLAN.md`, and `STATUS.md`.
Use `../../references/nova-delivery-checks.md` when filling or critiquing the
project artifacts.

## Requirements

Required skills:

- `project-creation`
- `prd` when the project needs a real PRD

Agent actions:

- define the problem, scope, non-goals, and acceptance criteria
- identify the owning components
- decide whether `PRD.md` is required or `PRD: N/A` is the correct choice
- if needed, create or update `PRD.md`
- record affected Nova surfaces and authoritative docs explicitly
- record required links for PRD, architecture docs, and implementation plan
- create the project folder and seed the artifacts if they do not exist

Required artifact updates:

- `PRD.md` when applicable
- `PROJECT.md`
- `STATUS.md`

Exit criteria:

- acceptance criteria are explicit
- in-scope and out-of-scope are explicit
- the PRD link is either real or `N/A` with a reason
- affected Nova surfaces and authoritative docs are explicit
- required links section exists and no entry is implicit
- current lifecycle stage is recorded

Escalate if:

- requirements are ambiguous enough that architecture or implementation would be
  guesswork

## Architecture

Required skills:

- `project-creation`
- `design` when architecture docs need to be created or updated
- `review-nova` before human feedback

Agent actions:

- decide whether existing architecture docs are sufficient
- if not, create or update the required architecture docs
- run an AI critique pass on the resulting draft before human review
- record authoritative docs and locked design decisions in `PROJECT.md`
- record affected Nova surfaces and contract boundaries explicitly
- identify shared semantic families, divergence points, and source-of-truth
  boundaries

Required artifact updates:

- `PROJECT.md`
- architecture docs when needed
- `STATUS.md`

Exit criteria:

- authoritative architecture source is linked
- affected Nova surfaces and contract boundaries are explicit
- missing-data behavior and forbidden fallbacks are clear
- the draft has an explicit critique verdict before human feedback
- any required architecture doc updates are done or an explicit blocker exists

Escalate if:

- semantics, contracts, or interfaces are changing but no authoritative design
  source is available

## Implementation Plan

Required skill:

- `project-creation`
- `review-nova` before human feedback

Agent actions:

- break the project into milestones or commit-sized increments
- identify owning components and cross-command impact for each milestone
- define proof commands and negative validation for each milestone
- run an AI critique pass on the plan draft before human review
- record blockers, dependencies, and open questions

Required artifact updates:

- `PLAN.md`
- `PROJECT.md` implementation plan link
- `STATUS.md`

Exit criteria:

- the next coding step is unambiguous
- milestone boundaries are explicit
- required validation is defined in command-level terms, not just directories
- affected Nova surfaces and proof commands are explicit
- the plan has an explicit critique verdict before human feedback

Escalate if:

- the plan cannot be sequenced without unresolved architectural ambiguity

## Implementation

Required skills:

- `coding`

Conditional skills:

- `fix-test` when working through test failures
- `building`
- `docs`

Agent actions:

- implement the current milestone
- follow source-of-truth, runtime-evidence, and semantic-family rules
- update status as reality changes
- split work into safe commit-sized increments when useful

Required artifact updates:

- `STATUS.md`
- `PLAN.md` when milestone sequencing changes

Exit criteria:

- milestone code is in place
- current blockers are explicit
- next validation target is clear

Escalate if:

- the implementation requires violating source-of-truth or architecture rules
- another component must be fixed first

## Validation

Required skills:

- `regressions`
- `building`

Conditional skill:

- `debug-nova`

Agent actions:

- map each acceptance criterion to concrete proof
- run the required commands
- verify all affected command surfaces
- include negative validation for missing-data or invalid states that should fail
  loudly
- use runtime evidence for reproducible runtime failures when needed
- record remaining gaps explicitly

Validation should include:

- acceptance criteria to proof mapping
- exact commands
- cross-surface checks
- negative validation
- build or environment prerequisites
- regression coverage added
- manual checks when automation is insufficient
- remaining gaps

Required artifact updates:

- `STATUS.md`
- `PROJECT.md` if acceptance criteria or blockers changed

Exit criteria:

- each in-scope acceptance criterion has proof or an explicit blocker
- known gaps are recorded rather than implied

Escalate if:

- validation cannot prove correctness because the current stage is really still
  architecture or implementation-plan work

## Commit And Publish

Not a project-creation stage. Use `checkin` after a validated milestone or
when the user explicitly asks for commit or push.
