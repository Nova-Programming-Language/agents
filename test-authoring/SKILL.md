---
name: test-authoring
description: Write or update Nova tests with the correct form, annotations, and validation flow; use when adding unit, program, or diagnostic coverage in the Nova repo.
---

# Test Authoring

## When to Use

Use this skill when a Nova change needs new tests, updated annotations, or a
decision about where a test belongs under `tests/` or `examples/`.

## Source of Truth

Read these in order before choosing a test strategy:

1. `docs/specs/nova-testing-spec.md`
2. `docs/architecture/test-system.md`
3. `docs/guides/tests.md`
4. `../references/artifact-freshness.md` when compiled or runtime-linked
   validation is in scope

The spec defines language-level test semantics. The architecture and guide
define how `nova test` classifies files, records cases, and treats annotations.

## Choose the Right Test Surface

Use the smallest test surface that proves the behavior:

- `diagnostic` for expected compiler failures and message checks
- `unit` for `struct ... implements UnitTest` behavior and helper methods
- `program` for whole-program `fn main()` execution
- `examples/` for end-to-end examples and public workflows, not as a
  substitute for focused unit or diagnostic coverage

Typical placement:

- `tests/compile_errors/` for diagnostic cases
- `tests/unit/` for `UnitTest`-based runtime cases
- `tests/build/` or targeted directories for program execution
- `examples/**` for e2e-facing example programs

Tier defaults matter:

- `examples/**` and `tests/e2e/**` default to `tier=e2e`
- everything else defaults to `tier=standard`
- `# E2E` and `# STANDARD` override the path default

## Required Annotation Rules

Load `references/placement-and-annotations.md` when you need the quick
placement and annotation matrix.

Core rules:

- `# ERROR` and `# ERROR: pattern` are for diagnostic tests only
- `# PENDING-ERROR` is for known-future diagnostics and is tracked, not
  enforced
- `# SKIP` and `# PANICS` apply to the next `test_*` method
- `# expect-output:` applies to the next `test_*` method and its following
  `#` lines define stdout exactly
- `# REQUIRES: --coverage` and `# REQUIRES: --coverage=branch` request
  coverage instrumentation for that file
- `# SKIP_COMPILED: reason` and `# COMPILED_ONLY` control compiled-run
  behavior

Do not rely on removed operational semantics like `tests/aspirational` as a
special regression bucket. Use explicit annotations instead.

## Authoring Checklist

1. Add or update success-path coverage.
2. Add or update invalid-usage or rejected-syntax coverage when relevant.
3. Cover edge and failure behavior, not only the happy path.
4. Keep skipped tests parseable and type-checkable; skip is not a syntax
   escape hatch.
5. If a file mixes `fn main()` and `UnitTest`, expect multiple case records.

## Validation Workflow

Run the narrowest command that proves the new test:

```bash
cargo run -q -p nova-cli -- test tests/unit/path_or_file.nova --skip-compiled
cargo run -q -p nova-cli -- test tests/compile_errors/path_or_file.nova
cargo run -q -p nova-cli -- test tests/build/path_or_file.nova
```

Use compiled differential validation when the change can diverge between the
interpreter and C backend:

```bash
cargo run -q -p nova-cli -- test tests/unit/path_or_file.nova --compiled
```

Do not trust compiled differential results after source edits unless the
runtime-linked artifacts were refreshed first. Use
`../references/artifact-freshness.md` to decide whether `nova-async-rt` and
`runtime` need a rebuild before trusting `--compiled` output.

For broader confirmation, hand off to the regression workflow skill and use
the canonical suite entrypoint:

```bash
scripts/full-regression.sh check
```
