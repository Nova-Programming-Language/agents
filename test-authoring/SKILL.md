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
6. Test changes must not introduce Rust or Nova test-runner warnings on the
   affected crate/test surface. Fix unused helpers, unreachable branches,
   stale imports, and annotation warnings instead of leaving them for later.

## Validation Workflow

Run the narrowest command that proves the new test:

```bash
target/release/nova test tests/unit/path_or_file.nova --backend interpreter
target/release/nova test tests/compile_errors/path_or_file.nova
target/release/nova test tests/build/path_or_file.nova
```

For normal targeted or directory `nova test` runs, rely on the default
exact-scope report and artifact recording unless you need an explicit export or
override.

Build the release CLI first if it may be stale:

```bash
cargo build --release -p nova-cli
```

Use compiled differential validation when the change can diverge between the
interpreter and C backend:

```bash
target/release/nova test tests/unit/path_or_file.nova --backend c
```

Do not trust compiled differential results after source edits unless the
runtime-linked artifacts were refreshed first. Use
`../references/artifact-freshness.md` to decide whether `nova-async-rt` and
`runtime` need a rebuild before trusting `--backend c` output.

Execution mode matters: bare `nova test` runs interpreter plus compiled
differential; `--backend interpreter` is interpreter only; `--backend c` is
compiled only; legacy `--skip-compiled` / `--compiled` remain compatibility
aliases; `--failed --from-last` is exact-scope snapshot replay only.

For broader confirmation, hand off to the regression workflow skill and use
the canonical suite entrypoint:

```bash
scripts/run-nova-full-regression.sh check
```

Do not describe `nova test tests/` or `nova test examples/` as canonical full
regression; those are still scoped `nova test` runs.

When authoring Rust-side tests, use a warning-deny compile check for the
affected crate if ordinary test output could hide test-only warnings:

```bash
RUSTFLAGS="-D warnings" cargo test -p <crate> --no-run
```
