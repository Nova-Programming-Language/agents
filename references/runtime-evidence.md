# Runtime Evidence

Use this reference when a bug is reproducible at runtime and the next useful
fact is likely a live value or control-flow decision.

## Core Rule

For a reproducible runtime bug, do minimal code reading to identify the likely
subsystem, then gather runtime evidence before changing code or proposing a
fix.

Do not keep reading implementation files once the next useful fact is a live
value, branch decision, or dispatch result that can be observed directly.

## Required Questions

Before continuing to read code, answer these:

1. What exact live fact do I need next?
2. Can one breakpoint, variable inspection, or narrow trace answer it?
3. If yes, why am I still reading code instead of collecting that evidence?

If you cannot name the next fact, more code reading may be justified. If you
can name it, prefer evidence collection.

## Tool Choice

- Prefer LLDB when one breakpoint or a short stepping session can answer the
  question.
- Prefer narrow tracing when the behavior spans many iterations, async
  boundaries, or timing-sensitive control flow that LLDB would distort.
- Prefer broader code reading only when the problem is architectural,
  non-reproducible, or the likely runtime surface is still unclear.

## Practical Rule

Minimal code reading is for finding the debug surface. Runtime evidence is for
deciding the fix.

Common anti-pattern:

- read multiple implementation files
- infer what values are probably happening
- patch code based on that inference

Preferred pattern:

- identify the likely owning subsystem
- run the narrow reproducer
- inspect the live values or dispatch path
- then patch the code based on observed evidence

## Reporting Expectation

In user-facing summaries for runtime bug work, state:

- the reproducer used
- the evidence collected
- the key live fact that changed or confirmed your understanding
