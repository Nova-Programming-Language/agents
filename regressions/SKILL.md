---
name: regressions
description: Run targeted or full regression workflows; use when verifying behavior, tracking full-suite regressions across sessions, or using the project's pinned container.
---

# Regression Test

## When to Use

Use this skill when validating behavior after code changes, whether through
targeted tests, crate-local tests, or the canonical full regression workflow.

Test and regression commands, result locations, and failure-record formats
come from the project profile (`../references/project-profile.md`). Never
guess a test or regression command; if the profile lacks the section a step
needs, ask the user.

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
- For targeted or directory test runs, rely on the test runner's default
  result recording unless you need an explicit export or override (profile
  §Tests).
- The full-regression script (profile §Regression) is the canonical
  full-suite state and failure tracker.
- A flaky test is a bug. There is no such thing as a transient flake: a
  nondeterministic failure has a deterministic cause (race, timing assumption,
  ordering dependency, resource contention) and must be diagnosed and fixed
  like any other failure. Never rerun-until-green, never label-and-skip, and
  never treat a retry-pass as satisfying a "proceed only if no failures"
  gate. If the cause cannot be found this session, file an issue with the
  reproduction evidence (frequency, seeds, thread counts) — an unfixed flake
  is an open bug, not noise. Reproduce it with a stress loop (vary test
  parallelism, iterate the single test directly; how is in profile §Tests),
  then diagnose.
- A scope that selects nothing can exit zero and report zero tests run.
  Before treating a scoped run as a pass, or a failure as not reproducing,
  confirm the executed count is non-zero and roughly what you expected
  (profile §Tests says how the runner reports it).
- Start a full run (or any multi-hour command) detached so background-task
  kills cannot take it down, and watch it with the Monitor tool — see
  "Starting a Long Run Detached" in `references/full-regression.md` for the
  exact pattern and the two observed failure modes it avoids.
- Know what the full-regression exit status means. Profile §Regression states
  whether it is absolute (any failing case fails the run) or a comparison
  against the baseline, and how a missing baseline is treated.
- Never report a comparison verdict as an absolute one. "Clean versus
  baseline", zero new failures, and a stable-failing verdict all mean *nothing
  got worse*. None of them mean *nothing is failing*. Only the absolute failure
  total, and an exit status that follows it, answer that.
- Report both numbers, always: how many cases failed in total, and how many of
  those are new. Name the known-failing set rather than letting it disappear
  into "clean". A promoted baseline records existing failures as expected, so
  after a promotion they stop being visible in the *verdict* — but they are
  still failing.
- To ask whether anything got worse rather than whether anything failed, read
  the new-failure count from the comparison artifact (profile §Regression).
  Never infer it from the exit status.
- When fixing a heuristic, fallback, or missing-data bug, add the regressions
  `../references/source-of-truth.md`, "Validation Expectations", requires.
- After reproducing a runtime failure, pivot to the debugger (profile
  §Debugging) or narrow tracing instead of broad code reading unless the issue
  is clearly architectural.
- When the issue involves related constructs, add paired tests that prove the
  shared behavior and any intended divergence.

## Workflow

1. Decide whether the task needs targeted validation or canonical full
   regression.
2. Use `../references/artifact-freshness.md` to check whether the command
   requires a rebuild first.
3. Use `references/full-regression.md` for the matching command family.
4. When validating code changes where warnings could appear only in test
   targets, add a warning-as-error compile check of those targets (profile
   §Build).
5. After reproducing a runtime failure, use `../references/runtime-evidence.md`
   to decide whether the next step should be the debugger or narrow tracing.
6. If you ran a full regression, use the regression tool's failure, run-list,
   and run-detail views (profile §Regression) before opening raw logs.
7. If related constructs are involved, use `../references/semantic-family.md`
   to decide which tests should be paired and where divergence should appear.
8. Add or update regressions for both the authoritative path and the
   missing-data path when relevant.

## Failure Interpretation

Keep the project's test-system model in mind (profile §Regression): what
identifies a test case canonically, which statuses count as failing, and
whether one test file may legitimately produce several cases (for example
across engines or forms).

The standard failure view is machine-first:

- use the regression tool's structured (JSON) failure view (profile
  §Regression) as the primary failure command; always prefer it over a
  plain-text view
- each failure record carries the fields profile §Regression lists, typically
  a classification, case identity, status, message, likely owner, failing
  phase, per-case artifacts, and a rerun hint
- classifications are relative to the chosen comparison target, typically
  the baseline
- prefer per-case artifacts over grepping the full run log; the run log is
  archival

Use the structured records for all downstream work:

- **Fixing**: pass records to `fix-test` — classification, owner, and phase
  fields let it prioritize which failure to tackle first
- **Triaging**: group records by owner or phase to partition failures
  across components or subsystems before assigning work
- **Filing issues**: attach the relevant record to each issue in the
  project's tracker (profile §Repository) so the fixer has machine-readable
  context from the start
- **Diffing runs**: compare structured output across runs to distinguish new
  regressions from pre-existing failures

For scoped test runs, use what the test runner prints (profile §Tests) to
confirm the exact scope, where results were stored, and that the run was not
empty (see Non-Negotiable Rules). Replaying the last failures re-reads a
snapshot of that scope only, and a scoped run over the whole test tree is
still not canonical full regression.

## After Running Regressions

When failures are found:

- To fix them: use `fix-test` — one failure at a time, root-cause
  diagnosis, audit, commit per fix
- To track them externally: use `issues` — file one issue per distinct
  root cause with the structured failure data
- To investigate non-obvious failures: use `bug-report` for local
  investigation notes before filing

Do not leave failures untracked. Every unexpected failure should either
be fixed in this session or filed as an issue for a future session.

## Open These References As Needed

- `../references/artifact-freshness.md`
- `../references/source-of-truth.md`
- `../references/runtime-evidence.md`
- `../references/semantic-family.md`
- `references/full-regression.md`
- `../references/project-profile.md`
