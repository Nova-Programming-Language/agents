---
name: debug
description: Debug reproducible failures with the project's debugger and breakpoint maps; use for crashes, wrong values, or control-flow bugs. Not for writing up a bug (use issues).
---

# Debug

## When to Use

Use this skill when a bug is reproducible and a debugger can answer the
question faster than temporary prints. It is the default for deterministic
runtime failures, wrong values at a known program point, regressions on any
debug surface in profile §Debugging (an interactive shell, a compiled native
binary that needs source-line stepping, metadata/dispatch paths), and
targeted crash or panic triage.

Project facts (build commands, debugger entrypoints, debug surfaces,
breakpoint maps, memory-error tools) are in profile §Debugging; find the
profile per `../references/project-profile.md`. Reasons behind the rules are
in `references/rationale.md`.

Read, when it applies:

- `../references/source-of-truth.md` — the bug looks like fallback inference,
  silent missing-data handling, or disagreement between structured and derived
  answers
- `../references/artifact-freshness.md` — the reproducer exercises release or
  runtime-linked artifacts
- `../references/runtime-evidence.md` — deciding whether to keep reading code
  or pivot to a debugger or tracing
- `../references/failure-attribution.md` — before any fix, whenever the defect
  surfaced in a different component from the one that produced the bad value,
  or the session runs inside a milestone whose ownership of the failure is not
  yet established
- `references/memory-error-detection.md` — a compiled native binary crashes
  with SIGBUS/SIGSEGV in release paths, the fault is near free/release/atomic
  decrement code, or the crash disappears under the debugger (libgmalloc, and
  the `MALLOC_STRICT_SIZE`/CoreFoundation ARM64 pitfall)

## Source-of-Truth Triage

- Before changing code, answer the Required Questions in
  `../references/source-of-truth.md` in the debug session. Also check whether
  the data is present and correct at the point of use, and if absent, whether
  the bug is in the producer, installer, or consumer.
- Never conclude a debug session with "use a heuristic" when the real finding
  is that required source-of-truth data was missing.

## Attribution Before Fix

Before editing, answer:

1. Which component's contract does this violate, and where is that contract
   written?
2. Is the frame where it surfaced the frame that produced it? Walk back to the
   producer rather than fixing at the observation point.
3. If the owning component is out of scope for this session or milestone, what
   is the blocker, and what is the issue to file?

The rest of the procedure is in `../references/failure-attribution.md`. Avoid
the two failure modes specific to debugging:

- **Fixing at the breakpoint**: a guard, clamp, or re-derivation in the
  consumer because that is where the debugger stopped.
- **Fixing at the wrong repository**: a toolchain defect repaired by shaping
  package code around it, or the reverse. Fix it at the source; if the source
  is not in scope, report the blocker.

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
- Treat missing required data as a producer/install bug first, not as a signal
  to patch in fallback inference.
- Fix at the owning component, not at the frame where the value was
  observed. When the two differ, say so explicitly in the report.
- Land the regression test per `../references/failure-attribution.md`,
  "Validation Expectations".

## Open These References As Needed

- `../references/source-of-truth.md`
- `../references/artifact-freshness.md`
- `../references/runtime-evidence.md`
- `../references/failure-attribution.md`
- `references/lldb-recipes.md`
- `references/breakpoints.md`
