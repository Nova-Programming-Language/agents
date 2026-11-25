# Why the Fix-Test Loop Works This Way

Reasons behind rules in `../SKILL.md`. The rules themselves live there.

- **One failure at a time, committed before the next.** Each commit is a
  clean, revertible checkpoint per fix: if a later fix goes wrong, earlier
  fixes are already safely committed.
- **The record's owner field is a hypothesis.** It says where the failure
  surfaced. Which component's contract it violates often differs, and
  accepting the owner field as the answer is how fixes land in the wrong
  layer.
- **Reproduce with a non-zero run count.** A reproduction that selects
  nothing looks identical to one that passes.
- **Common Change Patterns in diagnosis.** A test failure often means one
  component was updated but a co-required component was not.
- **Name the missing boundary fixture.** Without a test on both sides of a
  seam, the next failure there will be argued rather than located.
- **File discovered bugs instead of fixing them inline.** Fixing them inline
  is scope creep; filing them gives each its own fix cycle.
- **Verify the component, not only the test.** A fix that passes one test but
  breaks the component elsewhere is not a fix.
- **Never skip the audit for a small change.** Small changes are where
  shortcuts hide.
- **Fix at the owning component.** Compensating downstream for an upstream
  defect trades one defect for two and hides the first.
