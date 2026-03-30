# Structural Checks

Use this checklist when evaluating the structural dimension of an audit.
For each category, examine the diff and the surrounding code.

## Architecture

- Are changes in the correct layer? (e.g., not putting business logic in
  a presentation layer, not putting I/O in a pure computation module)
- Does the component that owns this behavior match what PROJECT.md says?
  If no PROJECT.md, does it match the implicit ownership in the codebase?
- Are module/package boundaries respected? Does the change introduce
  cross-boundary dependencies that did not exist before?
- If new files were created, are they in the right directory with the
  right naming convention?
- Does the implementation fulfill the architectural intent of the
  milestone, or does it achieve functional correctness through shortcuts
  that bypass the intended architecture? (e.g., hardcoding values instead
  of using the specified data source, using direct access instead of the
  required interface, keeping a legacy path alive instead of migrating)
- If the milestone's purpose is architectural migration or cleanup, verify
  that the old path is actually removed or reduced — not just that the
  new path exists alongside it.

## Abstraction

If BRIEF.md contains an Abstraction Context section, verify each
requirement against the implementation. If the brief has no abstraction
context, apply the general checks below — but flag the missing section
as a brief gap.

### Verify brief requirements

For each item in the brief's "Abstraction requirements for this
milestone":

- **Must use**: did the implementation call the specified abstraction,
  or did it reimplement the behavior? Grep for the abstraction's usage
  in the new code.
- **Must extend**: was the abstraction extended to cover the new case,
  or was a parallel path added alongside it?
- **Must not duplicate**: does the diff introduce a second
  implementation of the same concept? Check for functions with similar
  signatures, matching logic, or overlapping responsibility.
- **Must fix**: if the brief scoped fixing an incomplete or leaky
  abstraction, verify the fix — check that callers no longer do extra
  work or depend on internals.

### Missing abstractions

- Is there code in the diff that is semantically similar to existing
  code elsewhere in the touched files or their immediate neighbors?
  - Same operation on different types → should share a generic/trait/protocol
  - Same sequence of steps with minor variation → should share a function
    with parameters for the varying parts
  - Same data transformation in multiple places → should be one function
    called from each site

### Incomplete abstractions

- Does the diff add pre-processing, post-processing, or conditional
  bypasses around an existing abstraction? This suggests the
  abstraction doesn't cover the caller's actual need.
- Do multiple callers of the same abstraction each do the same extra
  work (unwrap-then-rewrap, nil-check-then-call, format-then-pass)?
  The repeated work should be inside the abstraction.

### Leaky abstractions

- Does the diff access struct fields, internal state, or
  representation details of a type that has a public interface? Callers
  should use the interface, not reach through it.
- Does the diff match on or branch over implementation details (type
  tags, internal enum variants, string representations) that could
  change without the caller knowing?

### Duplicative abstractions

- Does the diff introduce a new function, trait, or module that
  overlaps with an existing one? Check for:
  - Similar names in different modules
  - Functions with the same input/output shape and comparable logic
  - Two modules that both claim to "handle" the same concept

### Over-abstraction

- Abstractions with a single call site that add indirection without
  reuse or clarity are premature.
- Helper functions or utilities created for one-time operations that
  would be clearer inline.

## Semantic Families

- If PROJECT.md defines semantic families (constructs that should share
  a code path), does the implementation keep them unified?
- Does the diff introduce a new special case for something that should
  go through a shared path?
- If constructs are split into separate paths, is the divergence justified
  by genuinely different observable semantics (not just syntax differences)?

## Patterns

- Does the new code follow the conventions already established in the
  files it modifies? Look at:
  - Error handling style (Result vs exceptions vs error codes)
  - Naming conventions (casing, prefixes, verb forms)
  - Control flow patterns (early return vs nested, match vs if-else)
  - Data access patterns (direct field access vs accessors)
  - Test organization (setup, assertion style, naming)
- If the new code introduces a different convention, is there a reason,
  or is it inconsistency?

### Naming quality

- Are there generic names (`data`, `result`, `tmp`, `val`, `info`,
  `manager`, `handle`, `process`) that should be more specific? Each
  name should tell the reader what the thing *is* or *does* without
  reading the surrounding code.
- Do function names accurately describe what the function does? A
  function named `validate` that also transforms its input is
  misleading. A function named `do_thing` says nothing.
- Are boolean variables/parameters in predicate form (`is_X`, `has_X`,
  `should_X`)? Not `flag`, `status`, `check`.
- Is naming granularity consistent? If peer functions are `resolve_type`
  and `resolve_binding`, a new function called
  `get_and_validate_all_pending_items` is inconsistent.
- If the brief flagged naming inconsistencies in the existing code, did
  the implementation perpetuate or fix them?

### Comment quality

- Are there comments that restate the code? (`// set x to 5` above
  `x = 5`) — these add noise and go stale. Flag for removal.
- Do non-obvious code paths have a *why* comment? Magic numbers,
  workarounds, unusual control flow, and safety invariants that the
  type system doesn't enforce should be explained.
- Are there stale comments that no longer match the code they sit
  next to? Check whether comments near changed lines still apply.
- Does the diff introduce TODO/FIXME comments? New code should not
  contain TODOs — either do the work or file an issue.
- Are comments placed near the code they explain, or are they
  separated by enough lines to drift out of sync?

## Function Size and Complexity

Agents tend to produce monolithic functions that grow unchecked because
no one is there to say "this is getting too long." Check every function
that is new or significantly modified in the diff.

- Does the function exceed ~50 lines? Length alone is not a hard rule,
  but functions beyond this threshold almost always mix concerns. Read
  the function and identify whether it handles multiple distinct
  responsibilities (e.g., validation + transformation + persistence,
  or parsing + dispatch + error recovery).
- Does the function have 3+ levels of nesting? Deep nesting signals
  that conditional logic, iteration, and business logic are tangled.
  Inner blocks often deserve their own named function.
- Does the function use comments like "--- step 2 ---" or blank-line
  sections to separate phases? These are the function telling you it
  wants to be decomposed. Each labeled phase is a candidate for
  extraction.
- If the brief's implementation approach described the work as multiple
  steps, does each step map to its own function? A brief step that
  said "validate input" and "transform records" should not both live
  in a single function.
- Does the function have more than 4-5 parameters? High parameter
  count often means the function is doing too much or its inputs
  should be grouped into a struct/type.

When flagging, recommend the decomposition — name the responsibilities
and where the splits should go. Do not just say "function is too long."

Exception: match/switch statements with many arms that each do minimal
work (1-3 lines per arm) can legitimately be long. The concern is
mixed responsibilities, not line count alone.

## Completeness

This section catches the most common agent failure mode: doing the easy
parts of a milestone, then rationalizing the hard parts away.

- Compare the diff against every item in BRIEF.md's acceptance criteria,
  implementation approach, and done-when conditions. Is anything from
  the brief missing from the implementation?
- If the brief listed files to modify, were they all modified? If any
  were skipped, does the implementation actually work without those
  changes, or was scope silently reduced?
- If the milestone's purpose was architectural (migration, cleanup,
  interface change), verify the implementation actually achieves the
  architectural goal — not just that the tests pass. Functional
  correctness through shortcuts is not the same as completion.
- Check STATUS.md for any items the implementation agent moved to
  "Done" versus what was actually done. If it claims something is
  complete, verify the claim against the diff.

Anti-deferral rules:

- The implementation agent cannot descope items from the brief. If the
  brief says "do X, Y, Z" and the agent only does X and Y, the audit
  must flag Y as missing even if X and Y work perfectly.
- If the implementation agent recorded a blocker in STATUS.md and
  stopped, that is legitimate — but the blocker must be real (verifiable
  from the code or environment), not a judgment call like "this part
  doesn't make sense to do now" or "this would be better in a later
  milestone."
- The audit agent must not accept "deferred to next milestone" as a
  resolution for in-scope work. If something was in the brief, it is
  in scope. The only legitimate reasons for incomplete work are:
  1. A verifiable technical blocker (dependency not available, upstream
     bug, environment limitation) — recorded with evidence.
  2. The user explicitly descoped it during the session.
  Nothing else justifies partial completion.
- If the implementation appears to "work" but achieves correctness
  through a mechanism different from what the brief and plan specified
  (e.g., fallback to legacy path instead of using new path, hardcoded
  values instead of derived values, wrapping instead of migrating), the
  audit must flag this as an architectural shortcut, not a pass.

## Integrity

This section catches implementation dishonesty — code that appears to
work but hides problems, takes shortcuts, or papers over bugs.

### Shortcuts and heuristics

- Does the implementation use string matching, pattern heuristics, or
  hardcoded values where the brief or architecture specifies a proper
  data source, lookup, or computation?
- Are there magic numbers, special-case branches, or conditional checks
  that exist only to make specific tests pass without addressing the
  underlying requirement?
- Does the code use `unwrap()`, `force`, `!`, unchecked casts, or
  similar force-unwrapping where the brief or architecture expects
  proper error handling?
- Are there TODO/FIXME/HACK comments that acknowledge incomplete work
  while the implementation agent claims completion?

### Silent failures

- Does the code swallow errors, catch-and-ignore exceptions, return
  default values on failure, or use empty fallback branches where the
  brief or architecture requires loud failure?
- Are there `_ =>` / `default:` / `else` catch-all branches that
  silently handle cases that should be explicit?
- Does the code log errors but continue execution where it should
  propagate the error or abort?
- Are there conditions that can never be true in the current code but
  exist to suppress compiler warnings or linter errors about unhandled
  cases?

### Bug cover-ups

- Does the diff add workarounds for bugs rather than fixing them?
  (e.g., adding a nil check for a value that should never be nil,
  clamping a value that should never be out of range, retrying an
  operation that should succeed the first time)
- Does the implementation wrap existing buggy code to hide its output
  rather than correcting the bug?
- Are there "defensive" checks that paper over upstream contract
  violations instead of fixing the contract or failing loudly?
- If the diff modifies test expectations or assertion values, verify
  the new expectations are correct — not just that they match the
  (possibly wrong) implementation output.

### Shallow fixes

This catches the most common agent failure: touching many problems
without fully fixing any of them.

- For each fix in the diff, can you identify the root cause? If the
  diff changes behavior without a clear causal chain from symptom to
  root cause to fix, the fix is likely shallow.
- Does the fix address why the problem occurs, or does it just suppress
  the symptom? (e.g., catching an exception vs. preventing the
  condition that throws it)
- Were fixes verified independently? If the diff contains multiple
  independent fixes, each should have been tested on its own. Look
  for signs of batching — multiple unrelated changes in one commit
  where some work and some don't.
- If test failures were being fixed, does each fix name the root cause
  in STATUS.md or the commit message? "Made the test pass" without
  naming why it was failing is a red flag.
- Were test expectations modified? If so, verify the new expectations
  are correct — not just that they match the current (possibly wrong)
  output.

### Incomplete error paths

- Does every new error condition have a corresponding test or validation
  command in the brief?
- Are error messages specific enough to diagnose the failure, or are
  they generic ("an error occurred", "invalid input")?
- If the brief specifies negative validation (what should fail), verify
  those failure modes actually produce the expected errors.

## Refactoring Integrity

This section applies when the diff shows code removed from one component
and added to another — migrations, extractions, consolidations, and
moves. The most common agent failure in refactoring is deleting the old
code cleanly while only partially recreating it in the new location.

### Behavioral inventory verification

- Does BRIEF.md contain a behavioral inventory section? If the milestone
  is a refactoring and the inventory is missing, flag it — the brief
  was incomplete and the implementation cannot be verified against it.
- For each item in the behavioral inventory, verify the behavior exists
  in the new location. "Exists" means the code path is present and
  reachable, not just that a similarly-named function was created.
- If an inventory item is marked "no existing test coverage", pay extra
  attention — these are the behaviors most likely to be silently dropped.

### Deletion vs addition balance

- Identify the major code blocks deleted in the diff. For each one,
  locate the corresponding addition. If a deleted block has no
  corresponding addition anywhere in the diff, flag it.
- "Corresponding" means functionally equivalent — not necessarily
  identical code, but the same behavior or capability is preserved.
  Structural changes (different module, different interface) are fine
  as long as the behavior survives.
- If functionality was intentionally removed (not moved), the brief
  must say so explicitly. Unexplained deletions are a finding.

### Caller and integration verification

- If the old code had callers, verify those callers now use the new
  location. A moved function with no updated call sites is dead code
  — the behavior was effectively dropped even though the code exists.
- If the refactoring changed an interface (different function signature,
  different module path), verify all consumers were updated.

## Scope

- Were files modified that are not listed in BRIEF.md (or the stated
  scope of the change)?
- If yes, are the extra changes necessary for the stated goal, or are
  they drive-by refactors, formatting changes, or unrelated fixes?
- Drive-by improvements should be flagged — they make the diff harder
  to review and may introduce unrelated risk.
