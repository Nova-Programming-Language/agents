# Why Debugging Works This Way

Reasons behind rules in `../SKILL.md`. The rules themselves live there.

## Attribution before fix

A debugger answers *where the wrong value became observable*. That is not the
same as *which component owns the defect*, and the gap between them is where
abstraction violations get introduced: the breakpoint that finally showed the
bad value is the most tempting place to correct it, and usually the wrong one.

- **Fixing at the breakpoint** leaves the producer emitting the bad value, and
  every other consumer keeps receiving it.
- **Naming the owner in the report** when it differs from the observed frame
  tells the next reader whether the defect is closed or merely masked.
