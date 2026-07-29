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
- `../references/artifact-freshness.md`
- `references/build-recipes.md`

## Non-Negotiable Rules

- Decide which command you are about to run before deciding what to rebuild.
- Prefer `target/release/nova ...` for normal Nova CLI work. Treat
  `cargo run -p nova-cli -- ...` as a debug-CLI path, not the default.
- Never trust `target/release/nova` after source edits unless you just rebuilt
  it.
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
2. Use `../references/artifact-freshness.md` to identify the artifacts that
   command consumes.
3. Use `references/build-recipes.md` for the matching host, release, or
   container rebuild commands.
4. Run the validation command.
5. Report the rebuilt surfaces.

## Open These References As Needed

- `../references/artifact-freshness.md`
- `references/build-recipes.md`
