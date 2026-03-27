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

- Is there code in the diff that is semantically similar to existing code
  elsewhere in the touched files or their immediate neighbors?
  - Same operation on different types → should share a generic/trait/protocol
  - Same sequence of steps with minor variation → should share a function
    with parameters for the varying parts
  - Same data transformation in multiple places → should be one function
    called from each site
- Conversely, is anything over-abstracted? Abstractions with a single
  call site that add indirection without reuse or clarity are premature.
- Are helper functions or utilities created for one-time operations that
  would be clearer inline?

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

### Incomplete error paths

- Does every new error condition have a corresponding test or validation
  command in the brief?
- Are error messages specific enough to diagnose the failure, or are
  they generic ("an error occurred", "invalid input")?
- If the brief specifies negative validation (what should fail), verify
  those failure modes actually produce the expected errors.

## Scope

- Were files modified that are not listed in BRIEF.md (or the stated
  scope of the change)?
- If yes, are the extra changes necessary for the stated goal, or are
  they drive-by refactors, formatting changes, or unrelated fixes?
- Drive-by improvements should be flagged — they make the diff harder
  to review and may introduce unrelated risk.
