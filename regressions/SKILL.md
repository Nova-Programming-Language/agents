---
name: regressions
description: Run targeted or full Nova regression workflows; use when verifying behavior, tracking full-suite regressions across sessions, or using the pinned x86 container.
---

# Regression Test

## When to Use

Use this skill when validating Nova behavior after code changes, whether through
targeted tests, crate-local tests, or the canonical full regression workflow.

## Read First

1. `AGENTS.md`
2. `docs/guides/tests.md`
3. `docs/architecture/test-system.md`
4. `docs/specs/nova-testing-spec.md`
5. `../references/artifact-freshness.md`
6. `../references/source-of-truth.md` when the bug involves fallbacks or
   missing required data
7. `../references/runtime-evidence.md` for reproducible runtime failures
8. `../references/semantic-family.md` when the bug involves related constructs
9. `references/full-regression.md`

## Non-Negotiable Rules

- Prefer the smallest validation that proves the change.
- If the chosen command consumes release or runtime artifacts, refresh them
  first or use the canonical script that does it for you.
- `scripts/full-regression.sh` is the canonical full-suite state and failure
  tracker.
- When fixing a heuristic or fallback bug, add a regression that proves the
  authoritative path is used.
- When required data should exist, add a missing-data regression that fails
  loudly instead of passing through fallback behavior.
- After reproducing a runtime failure, pivot to LLDB or narrow tracing instead
  of broad code reading unless the issue is clearly architectural.
- When the issue involves related constructs, add paired tests that prove the
  shared behavior and any intended divergence.

## Workflow

1. Decide whether the task needs targeted validation or canonical full
   regression.
2. Use `../references/artifact-freshness.md` to check whether the command
   requires a rebuild first.
3. Use `references/full-regression.md` for the matching command family.
4. After reproducing a runtime failure, use `../references/runtime-evidence.md`
   to decide whether the next step should be LLDB or narrow tracing.
5. If you ran a full regression, use `failed`, `list`, and `show` before
   opening raw logs.
6. If related constructs are involved, use `../references/semantic-family.md`
   to decide which tests should be paired and where divergence should appear.
7. Add or update regressions for both the authoritative path and the
   missing-data path when relevant.

## Failure Interpretation

Keep the current test-system model in mind:

- canonical identity is `engine + form + tier + file + case_name`
- failing statuses are `fail`, `not_compilable`, `discovery_error`, and
  `infra_error`
- one file may legitimately produce multiple cases across engines/forms

The standard failure view is machine-first:

- use `scripts/full-regression.sh failed` before opening `full-run.log`
- `failed --json` emits the structured failure list
- each failure record includes `classification`, `case_id`, `engine`, `form`,
  `tier`, `file`, `case_name`, `source_case_key`, `status`, `message`,
  `owner`, `phase`, `artifacts`, and `rerun_hint`
- classifications are relative to the chosen comparison target, typically
  `baseline`
- prefer per-case `artifacts` over grepping `full-run.log`; the run log is archival

## After Running Regressions

When failures are found:

- To fix them: use `fix-test` — one failure at a time, root-cause
  diagnosis, audit, commit per fix
- To track them externally: use `issues` — file one GitHub issue per
  distinct root cause with the structured failure data
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
