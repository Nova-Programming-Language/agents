---
name: regressions
description: Choose and run targeted or full regression validation, read its results, and track full-suite state across sessions, including in the pinned container; use when verifying behavior or deciding which tests to run. Not for writing tests (use test-authoring) or fixing failures (use fix-test).
---

# Regression Test

## When to Use

Use this skill when validating behavior after code changes, whether through
targeted tests, crate-local tests, or the canonical full regression workflow.
Commands, result locations, and failure-record formats come from the project
profile (`../references/project-profile.md`). Never guess a test or regression
command; if the profile lacks the section a step needs, ask the user. Reasons
behind the rules are in `references/rationale.md`.

## Read First

1. The project's rules file (profile §Rules)
2. `../references/package-onboarding.md` when the target is a package repository
   or package directory
3. The project's test guide, test-system architecture doc, and testing spec
   (profile §Paths, §Tests)
4. `../references/artifact-freshness.md`
5. `../references/source-of-truth.md` when the bug involves fallbacks or
   missing required data
6. `../references/runtime-evidence.md` for reproducible runtime failures
7. `../references/semantic-family.md` when the bug involves related constructs
8. `references/full-regression.md`

## When to Run What

This section is the one home for which tests to run when; other skills point
here.

- Prefer the smallest validation that proves the change.
- Use targeted tests repeatedly while implementing.
- Run exhaustive tests (full regression and the other whole-component tiers in
  profile §Regression) only at the end of a milestone; between commits and
  checkpoints, run only the build and the tests that own the changed code.
- Run the owning component's suite once at the milestone gate.
- When a change touches shared code (utilities, data structures, common
  paths), also run the tests of the components that consume it.
- Run canonical/full regression only at the production-changing milestone
  gates PLAN.md defines and at final release, not after every file,
  checkpoint, or documentation-only milestone.

## Non-Negotiable Rules

- For package work, complete `../references/package-onboarding.md` before
  choosing or running the test command.
- If the chosen command consumes release or runtime artifacts, refresh them
  first or use the canonical script that does it for you.
- Treat new warnings on the validation surface as failures, per
  `../coding/SKILL.md`, "Implementation Rules".
- For targeted or directory runs, rely on the test runner's default result
  recording unless you need an explicit export or override (profile §Tests).
- The full-regression script (profile §Regression) is the canonical
  full-suite state and failure tracker.
- **A flaky test is a bug** with a deterministic cause (race, timing
  assumption, ordering dependency, resource contention). Reproduce it with a
  stress loop (vary test parallelism, iterate the single test directly; how is
  in profile §Tests), then diagnose and fix it like any other failure. Never
  rerun-until-green, never label-and-skip, and never let a retry-pass satisfy
  a "proceed only if no failures" gate. If the cause is not found this
  session, or is out of scope for the current fix, file an issue with the reproduction evidence (frequency, seeds,
  thread counts); an unfixed flake is an open bug.
- A scope that selects nothing can exit zero and report zero tests run.
  Before treating a scoped run as a pass, or a failure as not reproducing,
  confirm the executed count is non-zero and roughly what you expected
  (profile §Tests says how the runner reports it).
- Start a full run, or any multi-hour command, detached and watch it with the
  Monitor tool, per `references/full-regression.md`, "Starting a Long Run
  Detached".
- Know what the full-regression exit status means: profile §Regression says
  whether it is absolute or a comparison against the baseline, and how a
  missing baseline is treated.
- Never report a comparison verdict ("clean versus baseline", zero new
  failures, stable-failing) as an absolute one. Always report both the total
  number of failing cases and how many are new, and name the known-failing
  set (`references/full-regression.md`, "Comparison Verdict Versus Absolute
  Result").
- To ask whether anything got worse, read the new-failure count from the
  comparison artifact (profile §Regression), never from the exit status.
- When fixing a heuristic, fallback, or missing-data bug, add the regressions
  `../references/source-of-truth.md`, "Validation Expectations", requires.
- After reproducing a runtime failure, pivot to the debugger (profile
  §Debugging) or narrow tracing instead of broad code reading, unless the
  issue is clearly architectural.
- When related constructs are involved, add paired tests that prove the
  shared behavior and any intended divergence.

## Workflow

1. Decide between targeted validation and canonical full regression (When to
   Run What).
2. Check whether the command needs a rebuild first
   (`../references/artifact-freshness.md`).
3. Take the matching command family from `references/full-regression.md`.
4. When warnings could appear only in test targets, add a warning-as-error
   compile check of those targets (profile §Build).
5. After reproducing a runtime failure, choose the debugger or narrow tracing
   per `../references/runtime-evidence.md`.
6. After a full regression, use the regression tool's failure, run-list, and
   run-detail views (profile §Regression) before raw logs.
7. For related constructs, decide which tests to pair and where divergence
   should appear per `../references/semantic-family.md`.
8. Add or update regressions for the authoritative path and the missing-data
   path when relevant.

## Failure Interpretation

- Use the structured (JSON) failure view (profile §Regression) as the primary
  failure command, never a plain-text view, and prefer per-case artifacts
  over the run log. What the records hold is in `references/full-regression.md`,
  "Reading Failure Records".
- Use the records for all downstream work: pass them to `fix-test` to choose
  which failure to tackle first; group them by owner or phase to split
  failures across components; attach them to tracker issues; and compare them
  across runs to separate new regressions from pre-existing failures.
- For scoped runs, confirm from the runner's output (profile §Tests) the exact
  scope, where results were stored, and that the run was not empty. Replaying
  the last failures re-reads that scope's snapshot only, and a scoped run over
  the whole test tree is still not canonical full regression.

## After Running Regressions

Leave no unexpected failure untracked: fix it this session or file it.

- To fix: `fix-test` — one failure at a time, root-cause diagnosis, audit,
  commit per fix.
- To track: `issues` — one issue per distinct root cause, with the
  structured failure data.
- To investigate a non-obvious failure before filing: a local bug report per
  `issues`, "Local Bug Report".

## Open These References As Needed

- `../references/artifact-freshness.md`
- `../references/source-of-truth.md`
- `../references/runtime-evidence.md`
- `../references/semantic-family.md`
- `references/full-regression.md`
- `references/rationale.md`
- `../references/project-profile.md`
