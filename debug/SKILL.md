---
name: debug
description: Debug toolchain, runtime, REPL, compiled native binaries, or test failures with the project's debugger entrypoints and breakpoint maps; use for deterministic crashes, wrong values, source-line stepping, or control-flow bugs.
---

# Debug

## When to Use

Use this skill when a bug is reproducible and a debugger can answer the
question faster than adding temporary prints.

Project-specific facts (build commands, debugger entrypoints, debug surfaces,
breakpoint maps, memory-error tools) come from the project profile; see
`../references/project-profile.md` for how to find it. This skill names the
section, as "profile §Debugging".

This is the default skill for:

- deterministic runtime failures
- wrong values at a known program point
- regressions on any debug surface listed in profile §Debugging (for example
  an interactive shell, a compiled native binary that needs source-line
  stepping, or metadata/dispatch paths)
- targeted crash or panic triage

Read `../references/source-of-truth.md` when the bug looks like fallback
inference, silent missing-data handling, or disagreement between structured and
derived answers.
Read `../references/artifact-freshness.md` when the reproducer exercises
release or runtime-linked artifacts.
Read `../references/runtime-evidence.md` when you need to decide whether to
keep reading code or pivot to a debugger or tracing.
Read `../references/failure-attribution.md` before applying any fix, whenever
the defect surfaced in a different component from the one that produced the
bad value, or when the session runs inside a milestone whose ownership of the
failure is not yet established.
Read `references/memory-error-detection.md` when a compiled native binary
crashes with SIGBUS/SIGSEGV in release paths, the fault is near free/release/
atomic decrement code, or the crash disappears under the debugger. Covers libgmalloc
usage and the `MALLOC_STRICT_SIZE`/CoreFoundation ARM64 pitfall.

## Source-of-Truth Triage

Before changing code, answer these in the debug session:

1. What exact answer is the code trying to compute?
2. What source should authoritatively provide that answer?
3. Is that data present and correct at the point of use?
4. If it is absent, is the bug in the producer, installer, or consumer?

Do not conclude a debug session with “use a heuristic” when the real finding is
that required source-of-truth data was missing.

## Attribution Before Fix

A debugger answers *where the wrong value became observable*. That is not the
same as *which component owns the defect*, and the gap between them is where
abstraction violations get introduced — the breakpoint that finally showed the
bad value is the most tempting place to correct it, and usually the wrong one.

Before editing, answer:

1. Which component's contract does this violate, and where is that contract
   written?
2. Is the frame where it surfaced the frame that produced it? Walk back to the
   producer rather than fixing at the observation point.
3. If the owning component is out of scope for this session or milestone, what
   is the blocker, and what is the issue to file?

The rules that follow — including the milestone-versus-component split and the
forbidden compensating fixes — are in `../references/failure-attribution.md`.

Two failure modes are specific to debugging:

- **Fixing at the breakpoint.** Adding a guard, a clamp, or a re-derivation in
  the consumer because that is where the debugger stopped. The producer keeps
  emitting the bad value and every other consumer keeps receiving it.
- **Fixing at the wrong repository.** A defect in the toolchain repaired by
  shaping package code around it, or the reverse. Fix it at the source; if the
  source is not in scope, report the blocker.

A defect that a new caller exposes belongs to the component whose contract it
violates, not to the caller that reached it first. Exposure is not authorship.

## Workflow

1. Build a fresh debug target with the debug-build command in profile §Build.
2. If the reproducer exercises compiled/runtime-linked behavior, refresh the
   runtime artifacts first by following `../references/artifact-freshness.md`.
3. Choose the debug surface from the list in profile §Debugging: the stage
   that produces the bad data, the stage that executes it, or a compiled
   native binary when source-line stepping or external debugger breakpoints
   are the fastest evidence path.
4. Stop broad code reading once the next useful fact is a live value or branch
   decision.
5. Use `references/lldb-recipes.md` for generic debugger technique and the
   launch commands in profile §Debugging.
6. Use the breakpoint map in profile §Debugging (see `references/breakpoints.md`).
7. Inspect structured data before derived names or side tables.
8. Rerun the narrowest reproducer without the debugger after understanding the defect.

## Session Rules

- Prefer inspecting variables and stack frames over adding print statements.
- Confirm the exact command, working directory, and input file before stepping.
- Prefer the debugger when one breakpoint can answer the question.
- Prefer narrow tracing only when the bug spans many iterations, async
  boundaries, or timing-sensitive flow.
- For compiled binaries, build with the debug-symbols command in profile
  §Debugging and debug the produced executable. Respect the limits of that
  debug info as the profile states them.
- When the failure matches a failure type in profile §Debugging, inspect the
  values it lists for that type together.
- If structured data is supposed to answer the question, inspect that data
  before inspecting derived names or side tables.
- Treat missing required data as a producer/install bug first, not as a signal
  to patch in fallback inference.
- Fix at the owning component, not at the frame where the value was observed.
  When the two differ, say so explicitly in the report — the distinction is
  what tells the next reader whether the defect is closed or merely masked.
- Land the regression test in the owning component. A test that only pins the
  symptom at the surfacing site leaves the contract unproven.

## Open These References As Needed

- `../references/source-of-truth.md`
- `../references/artifact-freshness.md`
- `../references/runtime-evidence.md`
- `../references/failure-attribution.md`
- `references/lldb-recipes.md`
- `references/breakpoints.md`
