# AUDIT.md Section Format

AUDIT.md is append-only: each audit appends a dated `##` section and never
edits earlier ones. The file has one top-level heading, written only when it
is created: `# [Project slug] Audits`.

```md
## YYYY-MM-DD — [Milestone or change description]

- Base: `[commit]` ([caller | STATUS.md | BRIEF.md | PLAN.md | HEAD | merge-base])
- Scope: committed `[base]..HEAD` [+ uncommitted] [+ untracked: list]
- Depth: [focused | full] — [one-line reason]

### Verdict

- Result: [PASS | FAIL — failed dimension(s)]
- Truthful state: [scaffolded | implemented-but-not-exposed | exposed | release-verified]
- [Two or three sentences: what was audited and the key findings.]

### Findings

1. [Finding title]
   - evidence: `path/to/file:123`, `path/to/file:456`
   - [what is wrong]
   - commands: `[exact command that produced the evidence]`

### Completeness

- Brief items: [all accounted for | missing — list criteria, steps,
  done-when conditions, or files]
- Completion matrix: [all applicable cells proven | missing — list | N/A]
- Deferred items: [none | list, each with its blocker or user descoping]
- Placeholders: [clean | issues]

### Structural Assessment

[At focused depth, mark checks it skips as "not run (focused)".]

- Architecture: [clean | issues | unverified — doc gap] — docs used: [list]
- Repairs: [N/A | clean | issues — owning component vs. where the fix landed]
- Abstraction: [clean | issues]
- Semantic families: [unified | divergence]
- Patterns: [consistent | breaks]
- Integrity: [clean | issues]
- Scope: [clean | out-of-scope changes]

### Forward Impact

- [clean | risks — obstacles, wrong interfaces, tight coupling, decisions
  that may need undoing]
- Small fix now: [none | what would prevent later rework]

### Remediation Mapping

[Only when findings need action.]

- [Milestone or action]:
  - [what resolves finding N]
  - [owning component, if not the one audited]
```

## Notes

- Evidence is exact `file:line`, and commands are exact and re-runnable, so
  the orchestrator or a fix agent can verify each finding independently.
- Remediation Mapping is what the orchestrator copies into a fix agent's
  brief. A finding owned by another component or an earlier milestone still
  goes here, with its owner named — the milestone that exposed a defect does
  not inherit its fix.
- Earlier sections may be cited for progress ("finding 2 from YYYY-MM-DD is
  resolved") but never edited.
