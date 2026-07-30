# Structural Checks

Evaluate the diff and surrounding code across these categories.

## Architecture

Read the relevant `docs/architecture/` document before evaluating.

- **Component ownership**: does the diff put logic in the component that
  owns that behavior? Check **Invariant** and **Not responsible for**
  callouts. Code in a component marked "not responsible for" that behavior
  is in the wrong place.
- **Boundary correctness**: are layer, module, and directory boundaries
  respected? Flag new cross-boundary dependencies.
- **Change pattern completeness**: find the matching pattern in the
  architecture document's Common Change Patterns section. Verify every
  listed component was updated. Missing updates are the most common
  incomplete-implementation cause. Flag missing doc coverage.
- **Architectural intent**: does the implementation use the intended
  mechanism, or achieve correctness through shortcuts (hardcoding,
  direct access, keeping legacy paths)? For migration milestones, verify
  the old path is removed, not just supplemented.
- **Invariant preservation**: for each touched component, check the
  stated invariants still hold. Flag weakened invariants (e.g., adding
  fallbacks where the architecture says "fail loudly") even if tests pass.

## Abstraction

If BRIEF.md has an Abstraction Context section, verify each requirement.
If not, apply the checks below and flag the missing section.

- **Brief requirements**: for each must-use/must-extend/must-not-duplicate/
  must-fix item, verify compliance. Grep for the abstraction in new code.
- **Missing**: semantically similar code in the diff and neighboring files
  that should share a generic, function, or trait.
- **Incomplete**: callers doing extra work around an abstraction (unwrap-
  rewrap, nil-check-then-call, conditional bypass). The repeated work
  belongs inside the abstraction.
- **Leaky**: diff accesses struct fields, internal state, or representation
  details of a type that has a public interface.
- **Duplicative**: new function/trait/module that overlaps an existing one
  (similar names, same input/output shape, overlapping responsibility).
- **Over-abstraction**: single-call-site abstractions that add indirection
  without reuse or clarity. One-time helpers that would be clearer inline.

## Semantic Families

- Do shared-path constructs stay unified? Flag new special cases that
  should go through a shared path.
- If paths split, is the divergence justified by different observable
  semantics, not just syntax?

## Patterns

- Does new code follow established conventions (error handling, naming,
  control flow, data access, test organization)?
- **Naming**: flag generic names (`data`, `result`, `tmp`, `val`, `info`,
  `manager`), misleading names (function named `validate` that transforms),
  non-predicate booleans, inconsistent granularity. Check whether the
  brief flagged naming issues in existing code.
- **Comments**: flag restated-code comments, missing *why* comments on
  non-obvious paths, stale comments near changed lines, new TODO/FIXMEs.

## Function Size and Complexity

Check every new or significantly modified function:

- Exceeds ~50 lines with mixed concerns? Recommend decomposition.
- 3+ nesting levels? Inner blocks likely deserve named functions.
- Comment-delimited phases ("--- step 2 ---")? Each phase is a split
  candidate.
- Brief described multiple steps but they live in one function?
- More than 4-5 parameters? Inputs may need grouping.

Exception: long match/switch with minimal per-arm logic (1-3 lines) is
fine. The concern is mixed responsibilities, not line count alone.

## Completeness

Compare the diff against every BRIEF.md criterion, approach step, and
done-when condition. Flag anything missing or silently descoped.

For broad public contracts, inspect the completion matrix or build an
equivalent inventory covering success/failure behavior, engines/surfaces,
lifecycle, and packaging. A subset cannot pass the original criteria.

Classify the result as `scaffolded`, `implemented-but-not-exposed`, `exposed`,
or `release-verified`. Do not certify the first three as fully complete.

- If listed files were skipped, does the implementation actually work
  without them?
- If the milestone is architectural, verify the goal is achieved, not
  just tests passing.
- Verify STATUS.md "Done" claims against the diff.

Anti-deferral: agents cannot descope brief items. The only valid
incomplete-work reasons are a verifiable technical blocker (with evidence)
or explicit user descoping. "Deferred to next milestone" is not valid.
Architectural shortcuts (fallback to legacy, hardcoded values, wrapping
instead of migrating) are findings, not passes.

Search for `pass`, TODO/unimplemented markers, placeholder panics,
unconditional unsupported results, empty-success responses, ignored errors,
and test-only dispatch in production paths. Then inspect control flow
semantically: renamed or indirect placeholders evade text searches. A
placeholder is acceptable only when explicitly outside the committed public
contract and unreachable from it.

**Verification breadth**: targeted tests are appropriate during implementation;
the owning component's suite belongs at the milestone gate. Require
canonical/full regression only at defined production-changing gates and final
release. Do not demand repeated full runs after files or checkpoints.

## Integrity

Catches code that appears to work but hides problems.

- **Shortcuts**: string matching/heuristics/hardcoded values where the
  architecture specifies a proper source. Magic numbers or branches that
  only make specific tests pass. Force-unwrapping where proper error
  handling is expected. TODO/FIXME claiming completion.
- **Silent failures**: swallowed errors, catch-and-ignore, default-on-
  failure, catch-all branches that should be explicit.
- **Bug cover-ups**: workarounds instead of fixes (nil checks for never-
  nil values, clamping out-of-range values). Wrapping buggy code instead
  of correcting it. "Defensive" checks hiding upstream contract violations.
  Modified test expectations — verify they're correct, not just matching
  wrong output.
- **Shallow fixes**: no identifiable root cause in the diff's causal
  chain. Symptom suppression instead of prevention. Batched unrelated
  changes. Missing root-cause in STATUS.md or commit message.
- **Error paths**: new error conditions without tests. Generic error
  messages. Missing negative validation from the brief.

## Refactoring Integrity

Applies when the diff moves code between components. Refactoring should
result in a net reduction in lines of code. If the diff adds more than
it removes, flag it — the refactor is adding complexity, not reducing it.

- **Behavioral inventory**: if BRIEF.md has one, verify each item exists
  and is reachable in the new location. Extra attention to items marked
  "no existing test coverage." Flag missing inventory on refactoring
  milestones.
- **Deletion balance**: every major deleted block should have a
  corresponding addition. Intentional removals must be stated in the
  brief. Unexplained deletions are findings.
- **Caller updates**: moved code must have updated call sites. Changed
  interfaces must have updated consumers. Dead code at a new location
  (no callers) means the behavior was effectively dropped.

## Scope

- Were files outside BRIEF.md scope modified? Flag drive-by refactors,
  formatting changes, and unrelated fixes.
- Does the diff contain formatting-only changes to lines that were not
  otherwise modified? Flag as diff churn — it obscures real changes and
  pollutes git blame.
