---
name: debug-nova
description: Debug Nova frontend, interpreter, REPL, or test failures with repo-specific LLDB entrypoints and breakpoint maps; use for deterministic crashes, wrong values, or control-flow bugs.
---

# Debug Nova

## When to Use

Use this skill when a Nova bug is reproducible and a debugger can answer the
question faster than adding temporary prints.

This is the default skill for:

- deterministic runtime failures
- wrong values at a known program point
- REPL regressions
- metadata-install or dispatch bugs
- targeted crash or panic triage

Read `../references/source-of-truth.md` when the bug looks like fallback
inference, silent missing-data handling, or disagreement between structured and
derived answers.
Read `../references/artifact-freshness.md` when the reproducer exercises
release or runtime-linked artifacts.
Read `../references/runtime-evidence.md` when you need to decide whether to
keep reading code or pivot to LLDB or tracing.

## Source-of-Truth Triage

Before changing code, answer these in the debug session:

1. What exact answer is the code trying to compute?
2. What source should authoritatively provide that answer?
3. Is that data present and correct at the point of use?
4. If it is absent, is the bug in the producer, installer, or consumer?

Do not conclude a debug session with “use a heuristic” when the real finding is
that required source-of-truth data was missing.

## Workflow

1. Build a fresh debug target with `cargo build -p nova-cli`.
2. If the reproducer exercises compiled/runtime-linked behavior, refresh the
   runtime artifacts first by following `../references/artifact-freshness.md`.
3. Choose the debug surface:
   - frontend compile/validation for lowered-module production bugs
   - interpreter/runtime for post-compile wrong behavior
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
- When the failure is metadata-related, inspect the active unit, span, node ID,
  call-site entry, and symbol lookup result together.
- If structured data is supposed to answer the question, inspect that data
  before inspecting derived names or side tables.
- Treat missing required data as a producer/install bug first, not as a signal
  to patch in fallback inference.

## Open These References As Needed

- `../references/source-of-truth.md`
- `../references/artifact-freshness.md`
- `../references/runtime-evidence.md`
- `references/lldb-recipes.md`
- `references/breakpoints.md`
