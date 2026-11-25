# Artifact Freshness

Use this reference when deciding whether a validation command can trust the
artifacts it is about to execute. Commands and artifact names come from the
project profile (`project-profile.md`, §Build, §Regression, §Environments).

## Core Rule

Never assume existing build artifacts are fresh after source edits.

Preferred default:

- use the release CLI invocation from profile §Build for normal CLI work
- use the debug CLI only when you intentionally want it

Name the build profile, not a path. An invocation that resolves the build
output location itself keeps working where build artifacts are redirected; a
hardcoded artifact path does not exist in every checkout.

Before running validation, answer three questions:

1. What command am I about to run?
2. Which binary or libraries does that command actually execute?
3. Does the command rebuild those artifacts itself, or do I need to refresh them first?

## Artifact Matrix

Profile §Build should carry the project's concrete version of this matrix,
with an example command per row.

| Validation surface | CLI freshness | Runtime library freshness | Required action |
| --- | --- | --- | --- |
| Compile-only check | handled by the build tool | not relevant | none beyond the command itself |
| Debug CLI execution | handled by the build tool for the debug CLI | not guaranteed | debug-only; prefer the release CLI unless you are intentionally validating the debug binary |
| Component/unit tests | handled by the build tool for debug test binaries | not guaranteed | no manual CLI rebuild; rebuild runtimes only if the test depends on them |
| Release CLI execution | handled by the build tool for the release CLI | not guaranteed | the CLI is built as part of the command; runtime surfaces are not |
| Compiled/runtime-linked validation, debug CLI | handled by the build tool for the debug CLI | not handled automatically | debug-only; prefer the release CLI; rebuild the runtime libraries before trusting the result |
| Compiled/runtime-linked validation, release CLI | handled by the build tool for the release CLI | not handled automatically | rebuild the runtime libraries first |
| Canonical full regression (profile §Regression) | handled by the script | handled by the script | no extra manual rebuild needed |
| Pinned container validation (profile §Environments) | depends on container state | depends on container state | run the container prepare step before trusting stale container artifacts |

## Required Refresh Commands

Profile §Build lists the refresh commands for the release CLI and for the
runtime artifacts used by compiled/runtime-linked paths. Profile
§Environments lists the pinned container reset (prepare) command.

## Practical Rules

- For normal repo or package work, invoke the project CLI through the release
  invocation in profile §Build.
- An invocation that builds the CLI as part of the command means the CLI
  itself cannot be stale. It does not guarantee that the runtime libraries are
  fresh.
- The debug CLI builds a separate set of large debug artifacts, so do not use
  it as the routine path.
- A binary invoked by path is always suspicious after source edits unless you
  just rebuilt it. Resolve its location, or set the binary override the
  project scripts honor, as profile §Build describes.
- If compiled backend behavior, runtime ABI, or native linkage is in scope, refresh every runtime artifact surface before trusting the result.
- If you want the safest full-suite path, use the canonical full-regression script (profile §Regression) instead of hand-assembling release and runtime rebuilds.
- When reporting validation, state what was rebuilt or why the command itself guaranteed freshness.
