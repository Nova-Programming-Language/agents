---
name: debug-nova
description: Debug Nova frontend, interpreter, REPL, compiled C-backend native binaries, or test failures with repo-specific LLDB entrypoints and breakpoint maps; use for deterministic crashes, wrong values, source-line stepping, or control-flow bugs.
---

# Debug Nova

## When to Use

Use this skill when a Nova bug is reproducible and a debugger can answer the
question faster than adding temporary prints.

This is the default skill for:

- deterministic runtime failures
- wrong values at a known program point
- REPL regressions
- compiled C-backend behavior that needs source-line stepping or breakpoints
- metadata-install or dispatch bugs
- targeted crash or panic triage

Read `../references/source-of-truth.md` when the bug looks like fallback
inference, silent missing-data handling, or disagreement between structured and
derived answers.
Read `../references/artifact-freshness.md` when the reproducer exercises
release or runtime-linked artifacts.
Read `../references/runtime-evidence.md` when you need to decide whether to
keep reading code or pivot to LLDB or tracing.
Read `../references/failure-attribution.md` before applying any fix, whenever
the defect surfaced in a different component from the one that produced the
bad value, or when the session runs inside a milestone whose ownership of the
failure is not yet established.
Read `references/memory-error-detection.md` when a compiled Nova binary
crashes with SIGBUS/SIGSEGV in release paths, the fault is near free/release/
atomic decrement code, or the crash disappears under LLDB. Covers libgmalloc
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

1. Build a fresh debug target with `cargo build -p nova-cli`.
2. If the reproducer exercises compiled/runtime-linked behavior, refresh the
   runtime artifacts first by following `../references/artifact-freshness.md`.
3. Choose the debug surface:
   - frontend compile/validation for lowered-module production bugs
   - interpreter/runtime for post-compile wrong behavior
   - compiled C-backend native binary when source-line stepping or external
     debugger breakpoints are the fastest evidence path
4. Stop broad code reading once the next useful fact is a live value or branch
   decision.
5. Use `references/lldb-recipes.md` for launch commands.
6. Use `references/breakpoints.md` for the file/line map.
7. Inspect structured data before derived names or side tables.
8. Rerun the narrowest reproducer without LLDB after understanding the defect.

## Session Rules

- Prefer inspecting variables and stack frames over adding print statements.
- Confirm the exact command, working directory, and input file before stepping.
- Prefer LLDB when one breakpoint can answer the question.
- Prefer narrow tracing only when the bug spans many iterations, async
  boundaries, or timing-sensitive flow.
- For compiled Nova binaries, use `nova build --backend c --debug-symbols`
  and debug the produced executable. Treat this as line-table/source-location
  support for external debuggers, not as a full Nova variable debug-info model.
- When the failure is metadata-related, inspect the active unit, span, node ID,
  call-site entry, and symbol lookup result together.
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
