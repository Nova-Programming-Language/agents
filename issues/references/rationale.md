# Why Issues Work This Way

Reasons behind rules in `../SKILL.md`. The rules themselves live there.

- **One subsystem label, by where the defect is.** The label is how agents
  find work and how triage routes issues. A label that follows the symptom
  sends the next agent to the wrong component.
- **Paste structured failure records into the body.** The fixer has
  machine-readable context from the start. Several tests often fail from one
  cause, which is why records are grouped by owner and phase before filing.
- **Claims are comments, not assignment.** Several agents may share a single
  tracker account, so assignment cannot tell them apart; a comment with a
  unique session identifier can. The claim comment is the lock, which is why
  it comes before any reading of code.
- **Reassign with a diagnosis comment.** The next agent picks up where you
  left off instead of repeating the diagnosis. The label must reflect the
  current understanding of ownership.
- **Stale claims stay until the user releases them.** A claim with no
  follow-up may mean the agent crashed or its session ended, but another
  agent cannot tell that from a slow one.
- **Triage is a fast pass.** Reading titles, bodies and labels, not code, keeps it
  cheap; full diagnosis happens during fixing.
- **Prefer issues with more evidence.** An exact reproduction and file:line
  references make them faster to fix.
