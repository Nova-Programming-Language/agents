---
name: building
description: Build or rebuild the project's CLI, runtimes, or pinned container; use when compiling changes or refreshing stale artifacts. Not for running tests (use regressions).
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
- Invoke the CLI, resolve binaries, refresh runtime and container artifacts,
  and report what was rebuilt per `../references/artifact-freshness.md`
  (its Core Rule, Artifact Matrix, and Practical Rules).
- Treat new build warnings as failures, per `../coding/SKILL.md`,
  "Implementation Rules".

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
