# Structural Checks

Evaluate the diff and surrounding code across these categories. Which checks
run depends on the depth chosen per `scope-and-depth.md`.

**What to demand**: targeted tests during implementation; the owning
component's suite at the milestone gate; canonical/full regression only at
defined production-changing gates and final release. Do not demand repeated
full runs after files or checkpoints.

## Architecture

Find the governing doc for every changed path per
`../../references/architecture-docs.md` and record which docs you used. A doc
gap makes this section `unverified — doc gap`, never clean.

- **Component ownership**: does the diff put logic in the component that
  owns that behavior? Code in a component whose **Not responsible for**
  callout names that behavior is in the wrong place.
- **Boundary correctness**: are layer, module, and directory boundaries
  respected? Flag new cross-boundary dependencies.
- **Change pattern completeness**: find the matching pattern in the
  architecture document's Common Change Patterns section and verify every
  listed component was updated — missing updates are the most common cause
  of incomplete work. Flag a change with no matching pattern as missing doc
  coverage.
- **Architectural intent**: does the implementation use the mechanism the
  brief and plan specify? Hardcoded values, direct access, fallback to a
  legacy path, or wrapping instead of migrating are findings even when tests
  pass. For migrations, verify the old path is removed, not supplemented.
- **Invariant preservation**: for each touched component, check its stated
  **Invariant** callouts still hold. Flag weakened invariants (e.g., a
  fallback where the architecture says "fail loudly") even if tests pass.
- **Doc currency**: if the diff moves ownership, changes a boundary or
  invariant, adds a change type with no Common Change Pattern, or adds, moves,
  or deletes paths the architecture map covers, the governing doc or map is
  updated in the same change. A missing update is a finding.

## Repairs

Applies when the diff fixes a failure. Read the diagnosis (commit message,
STATUS.md, or BRIEF.md). Reasoning and wrong-layer patterns are in
`../../references/failure-attribution.md`.

- **Placement**: the fix sits in the component whose contract was violated,
  even if a fix elsewhere turns the test green. The tell: the diff changes a
  component the diagnosis never named.
- **Root cause**: the diff's causal chain reaches an identifiable root cause,
  recorded in STATUS.md or the commit message. Symptom suppression and
  batched unrelated changes are findings.
- **Class coverage**: name the rule the defect violated and list every site
  in the codebase (not just the diff) that enforces or depends on it. An
  uncovered site in the class is a finding; several independent
  implementations of one rule are a finding.
- **Re-audits**: check every site in the class, not only the ones flagged
  last time. If an earlier finding's class has come back in a new form,
  report **"no single home"** as a structural blocker — the remedy is design,
  not another site.
- **Regression test layer**: the test pins the violated contract in the
  owning component, not the symptom where it surfaced. If not, name the
  component it belongs in.
- **Recorded**: a fix to another component's defect has a diagnosis, issue,
  or declared-regression record. Absorbing it unrecorded is the finding, even
  if the repair is correct.

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

Constructs that share meaning at some layer (see
`../../references/semantic-family.md`).

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

- **Brief coverage**: every BRIEF.md acceptance criterion, approach step,
  done-when condition, and listed file is accounted for. If listed files
  were skipped, does the implementation work without them? Verify STATUS.md
  "Done" claims against the diff.
- **Completion matrix**: for broad public contracts, check every applicable
  cell of the plan's completion matrix (see
  `project-creation/references/plan-template.md`), or build an equivalent
  inventory: success/failure behavior, engines/surfaces, lifecycle,
  packaging. A subset cannot pass the original criteria. Classify the result
  using the truthful states in `../SKILL.md`.
- **No deferral**: agents cannot descope brief items. The only valid reasons
  for unfinished work are a verifiable technical blocker (with evidence) or
  explicit user descoping. "Deferred to next milestone" is not valid, and
  neither is a hold that rests on an unmeasured premise. Sizing trade-off:
  `../../orchestrate/references/brief-template.md`, "Size by risk and reward".
- **Placeholders**: search for `pass`, TODO/unimplemented markers,
  placeholder panics, unconditional unsupported results, empty-success
  responses, ignored errors, and test-only dispatch in production paths.
  Then read the control flow — renamed or indirect placeholders evade text
  search. A placeholder is acceptable only when outside the committed public
  contract and unreachable from it.

## Integrity

Catches code that appears to work but hides problems.

- **Shortcuts**: string matching, heuristics, or hardcoded values where the
  architecture specifies a proper source. Magic numbers or branches that
  only make specific tests pass. Force-unwrapping where proper error
  handling is expected. TODO/FIXME claiming completion.
- **Silent failures**: swallowed errors, catch-and-ignore, default-on-
  failure, catch-all branches that should be explicit.
- **Bug cover-ups**: nil checks for never-nil values, clamping out-of-range
  values, wrapping buggy code instead of correcting it, "defensive" checks
  hiding upstream contract violations. Modified test expectations — verify
  they're correct, not just matching wrong output.
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

- Files outside BRIEF.md scope: flag drive-by refactors and unrelated fixes.
- Formatting churn: flag formatting-only changes to lines not otherwise
  modified, and any formatter run beyond the files the task touched (e.g.
  package- or repo-wide; formatter scope is in profile §Formatting) unless
  explicitly required. Churn obscures real changes and pollutes git blame.
