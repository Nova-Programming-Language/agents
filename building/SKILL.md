---
name: building
description: Build or rebuild the project's CLI, runtimes, or pinned container environment; use when compiling local changes, refreshing stale artifacts, or preparing test and regression runs.
---

# Building

## When to Use

Use this skill when a repo task needs a rebuild, a clean toolchain state,
release artifacts, or the project's pinned container workflow.

Project commands, artifact locations, and environments come from the project
profile (`../references/project-profile.md`). Never guess a build command; if
the profile lacks the section a step needs, ask the user.

## Read First

- The project's build docs and rules file (profile §Paths, §Rules)
- `../references/package-onboarding.md` when building or preparing a package
  repository
- `../references/artifact-freshness.md`
- `references/build-recipes.md`

## Non-Negotiable Rules

- Decide which command you are about to run before deciding what to rebuild.
- Invoke the project CLI the way profile §Build says (its default build
  profile, and how to select a debug build only when you deliberately want
  one). Prefer an invocation that resolves the build output location itself
  over a hardcoded artifact path, which breaks when build output is redirected.
- Know what that invocation refreshes and what it does not: an invocation
  that builds the CLI cannot leave the CLI stale, but runtime libraries it
  links may still be stale (profile §Build lists them).
- If you do invoke a built binary by path, resolve its location the way
  profile §Build describes rather than assuming a default directory.
- Never trust compiled/runtime-linked validation after source edits unless the
  runtime artifacts were refreshed or a canonical script did it.
- Run the container prepare step (profile §Environments) before trusting stale
  container artifacts.
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
4. Use `references/build-recipes.md` to pick the build surface (host, release,
   runtime, full regression, or container), then take its commands from
   profile §Build or §Environments.
5. Run the validation command.
6. Report the rebuilt surfaces.

## Open These References As Needed

- `../references/artifact-freshness.md`
- `../references/package-onboarding.md`
- `references/build-recipes.md`
- `../references/project-profile.md`
