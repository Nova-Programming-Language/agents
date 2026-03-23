# Source of Truth

Use this reference when code needs an answer that could be obtained from more
than one place.

## Core Rule

When code needs an answer, it must use the authoritative source for that
answer. If that source is missing and should exist, treat it as a bug or
contract violation. Do not add heuristics, silent degradation, or fallback
inference from secondary data.

## Required Questions

Before implementing or changing lookup, dispatch, validation, or type-driven
behavior, answer these:

1. What exact answer is this code trying to compute?
2. What data source is authoritative for that answer?
3. If that data is missing, is that a valid state or a producer/install bug?
4. If it is a bug, which component owns producing or installing the data?

If you cannot answer these clearly, stop and inspect the contract or ask.

## Forbidden Patterns

- String-name heuristics when structured data exists or should exist.
- Pattern-matching on function names, type names, or symbol spelling to recover
  semantics.
- Fallback from source-of-truth contract data to AST walks, registries, or
  secondary maps when the contract should already provide the answer.
- `unwrap_or_default`, empty collections, `None => continue`, or silent early
  returns on required data.
- Best-effort inference that masks missing producer output instead of surfacing
  the defect.
- “Temporary” compatibility paths that leave the system accepting invalid or
  incomplete contract data without a loud failure.

## Required Behavior When Data Is Missing

- If the state is validly optional, handle it explicitly and document why.
- If the state should exist, fail loudly or return a diagnostic that identifies
  the missing required data.
- If another component owns the data, repair that producer/install path or
  report the blocker explicitly instead of guessing.

Silent downgrade is not an acceptable substitute for contract enforcement.

## Validation Expectations

- Add or update tests that prove the authoritative path is used.
- Add or update tests for the missing-data path when the data should exist.
- Missing required data should produce an explicit failure, validation error, or
  blocker signal, not a green result through fallback inference.

## Reporting Expectation

In user-facing summaries, state:

- the answer being computed
- the authoritative source used
- what happens if the source is missing
- which component owns the missing data if the work is blocked
