---
name: building
description: Build or rebuild the Nova CLI, runtimes, or pinned Linux/x86_64 container; use when compiling local changes, refreshing stale artifacts, or preparing test and regression runs.
---

# Building

## When to Use

Use this skill when a Nova repo task needs a rebuild, a clean toolchain
state, release artifacts, or the pinned Linux/x86_64 container workflow.

## Read First

- `BUILDING.md`
- `AGENTS.md`
- `../references/package-onboarding.md` when building or preparing a package
  repository
- `../references/artifact-freshness.md`
- `references/build-recipes.md`

## Non-Negotiable Rules

- Decide which command you are about to run before deciding what to rebuild.
- Invoke the Nova CLI by build profile, not by path:
  `cargo run --release -q -p nova-cli -- ...` for normal work, and
  `cargo run -q -p nova-cli -- ...` only when you deliberately want the debug
  CLI. Cargo resolves the target directory itself, so this keeps working when
  build artifacts are redirected elsewhere; a hardcoded `target/release/nova`
  does not.
- Running through Cargo makes the CLI current for that invocation, so it cannot
  be stale after a source edit. It does not refresh `nova-async-rt` or
  `runtime/libnova_runtime.a`.
- If you do invoke a built binary by path, resolve it rather than assuming
  `target/`: `cargo metadata --format-version 1 --no-deps` reports
  `target_directory`, and the repository scripts honor `NOVA_BIN`.
- Never trust compiled/runtime-linked validation after source edits unless
  `nova-async-rt` and `runtime` were refreshed or a canonical script did it.
- Use `scripts/nova-x86-container.sh prepare` before trusting stale container
  artifacts.
- Builds after code or test changes must be warning-free on the affected
  surface. Treat new compiler or runtime build warnings as failures unless an
  explicit blocker is recorded.
- In validation summaries, state what was rebuilt or why the command itself
  guaranteed freshness.

## Workflow

1. Name the exact command you are about to run.
2. For package work, complete `../references/package-onboarding.md` and honor
   package-specific prerequisites before building or preparing.
3. Use `../references/artifact-freshness.md` to identify the artifacts that
   command consumes.
4. Use `references/build-recipes.md` for the matching host, release, or
   container rebuild commands.
5. Run the validation command.
6. Report the rebuilt surfaces.

## Open These References As Needed

- `../references/artifact-freshness.md`
- `../references/package-onboarding.md`
- `references/build-recipes.md`
