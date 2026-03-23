# Test Placement And Annotations

## Quick Placement Map

- `tests/compile_errors/`: expected compiler diagnostics
- `tests/unit/`: `UnitTest` discovery and `test_*` methods
- `tests/build/`: executable `fn main()` programs
- `tests/e2e/**`: e2e tier by path default
- `examples/**`: e2e tier by path default and example-facing coverage

These are conventions, not the semantic source of truth. Engine and form come
from the file contents plus annotation semantics.

## Tier Overrides

- `# E2E` or `# E2E: reason` forces `tier=e2e`
- `# STANDARD` forces `tier=standard`
- using both in one file is a discovery error

## Annotation Placement Rules

Diagnostic:

- `# ERROR` goes on the line that should fail compilation
- `# ERROR: pattern` also requires a matching message substring
- `# PENDING-ERROR` marks expected-future diagnostics and does not fail the run

Unit and program:

- `# SKIP` or `# SKIP: reason` must be immediately before the next `test_*`
- `# PANICS` or `# PANICS: pattern` must be immediately before the next
  `test_*`
- `# expect-output:` must be immediately before the next `test_*`
- each following `#` line becomes one expected stdout line
- `# |` preserves leading whitespace in expected output
- `# REQUIRES: --coverage` and `# REQUIRES: --coverage=branch` apply to the
  file
- `# SKIP_COMPILED: reason` and `# COMPILED_ONLY` apply to compiled runs for
  the file

## Common Authoring Mistakes

- putting `# SKIP` on a helper method and expecting it to affect another test
- using `# ERROR` in a runtime test instead of a diagnostic file
- storing blocked work in a special suite instead of using `# SKIP` or
  `# PENDING-ERROR`
- forgetting that skipped tests still need to parse and type-check
- relying on directory name alone when the file content actually changes form
