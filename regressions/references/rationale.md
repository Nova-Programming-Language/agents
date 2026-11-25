# Why the Regression Rules Work This Way

Reasons behind rules in `../SKILL.md`. The rules themselves live there.

- **Exhaustive tests only at the end of a milestone.** Corpus walks and
  whole-crate or whole-CLI suites run after every commit made milestones take
  hours per small change. The build plus the tests that own the changed code
  catch what a commit can break; the milestone gate catches the rest.
- **There is no transient flake.** A nondeterministic failure has a
  deterministic cause, and passing on retry hides it, never disproves it. An
  unfixed flake is an open bug, not noise.
- **Check the executed count.** A scope that selects nothing looks identical
  to one that passes at a glance.
- **Detached long runs.** Background-task kills can take a long run down; the
  two observed failure modes are in `full-regression.md`, "Starting a Long Run
  Detached".
- **Both failure numbers, always.** A promoted baseline records existing
  failures as expected, so after a promotion they stop being visible in the
  verdict, but they are still failing. "Nothing got worse" and "nothing is
  failing" are different answers; only the absolute total answers the second.
- **Structured records first.** They carry enough context to classify,
  triage, and fix failures without parsing raw compiler output, and the
  fixer has machine-readable context from the start.
