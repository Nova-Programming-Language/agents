# Build Recipes

Use this file after the SKILL tells you which build surface you need. The
commands for each surface are in the project profile
(`../../references/project-profile.md`); this file says which surface to use
and what each one does and does not refresh.

## Fast Validation

A compile-only check of the affected components or modules (profile §Build). It
proves the code compiles; it executes nothing, so artifact freshness is not a
concern.

## Debug Builds

Debug build and unit-test commands (profile §Build). These keep debug
artifacts fresh. They do not refresh the release CLI or the runtime libraries
used by compiled/runtime-linked paths.

## Release Surfaces

- **Release CLI**: the release build command, and the preferred CLI
  invocation after it (profile §Build).
- **Runtime artifacts** used by compiled/runtime-linked paths: their rebuild
  commands (profile §Build).

An invocation that builds the CLI as part of the command cannot leave the CLI
stale. If the command exercises compiled/runtime-linked behavior, refresh the
runtime surfaces first -- the CLI invocation does not do that for you.

Use the debug CLI only when you intentionally want it, for example while
debugging the CLI's own behavior. It builds a separate set of large debug
artifacts.

## Canonical Full Regression

The full-regression command and its baseline command (profile §Regression).
Profile §Regression also says which artifacts that script rebuilds itself;
when it rebuilds the CLI and runtime libraries, no manual rebuild is needed
first.

## Pinned Container

The container lifecycle (build image, start, prepare, run regression) and
one-off exec/status commands are in profile §Environments.

Use the prepare step before trusting existing container binaries or runtime
artifacts.

## Quick Heuristics

- Change that touches no runtime or compiled-backend code (for example a
  frontend-only change): start with the compile-only check.
- Routine package or repo validation: prefer the release CLI invocation that
  builds the CLI as part of the command (profile §Build).
- Runtime or compiled-backend change: rebuild release CLI plus runtime
  libraries, or use the canonical full regression script.
- Release-path bug: do not trust debug-only validation.
- Suspicious stale behavior: rerun through the canonical script instead of
  guessing.
