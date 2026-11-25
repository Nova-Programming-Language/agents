# Why the Pipeline Works This Way

Reasons behind rules in `../SKILL.md`. The rules themselves live there.

## Scout before brief

A brief written from an unmeasured premise fixes its scope around a guess.
When the guess is wrong, the falsification arrives at implementation time,
after a brief, a worker run, and often an audit have been spent on it. A scout
measures the load-bearing facts first, so the brief is written from them.

## Checkpoints are not completion

`[checkpoint]` entries are written while the agent is still working. Seeing
one is confirmation the agent is making progress, not a reason to stop
waiting.

## Commit after audit

Committing each passing milestone gives the next milestone's audit a clean
base for its diff.

## Model choice

- **Audits and critiques on `fable`**: they are the highest-stakes judgement,
  and a model different from the implementer's gives a reading that isn't
  anchored on the implementer's reasoning.
- **Briefs on `fable`**: the brief fixes the representation, the single home
  of each rule, and the acceptance; errors there cause repeated audit
  failures. (User request, 2026-11-16.) The brief's critique still runs, so
  brief and critique are both on `fable`.
- **Implementation on `opus`**: long, multi-step code and spec work.
- **Mechanical work on `sonnet`**: the inputs are already written down; the
  task needs no deep judgement.

## Attribute before routing

Routing a failure without attributing it produces the two most expensive
outcomes available: a fix applied at the layer that surfaced the defect
rather than the one that owns it, and a milestone credited with work it did
not do. Attribution by opinion is not an acceptable substitute for a fixture.

## Entry and exit gates

Without recorded green points, "newly exposed" is an assertion instead of a
bisect, and nothing after the first red baseline can be attributed at all.

## Fix site

A downstream patch for an upstream defect trades one defect for two and
hides the first.

## Long-running verification

A sub-agent's processes, including anything it started with a background bash
call, are killed the moment it returns its final message. Only background
tasks started by the main session survive turn boundaries and emit completion
notifications. So a worker that backgrounds a run and returns "waiting for
results" has orphaned a dead process and made an invalid exit.

A continuation agent cannot recover this: a new Agent call does not inherit
the returned worker's context, and it would hit the same wall. The
orchestrator can recover it, because the diff is already in the working tree
and the worker's checkpoints record its verification claims.

## Authoritative state in files

An agent's return message is a summary, not the source of truth. It may be
truncated, ambiguous, or describe intermediate progress.
