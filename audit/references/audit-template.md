# AUDIT.md Section Format

AUDIT.md is append-only. Each audit appends a new dated section.
Do not overwrite prior sections.

```md
# [Milestone or change description] Audit

## YYYY-MM-DD

### Verdict

- [One-paragraph summary: what was audited, overall outcome, and key
  findings. State whether the milestone passes all three dimensions.]

### Findings

[Numbered list. Each finding includes evidence with exact file:line
references and commands used.]

1. [Finding title or description]
   - evidence:
     - `path/to/file.rs:123`
     - `path/to/file.rs:456`
   - [explanation of what is wrong or what was observed]
   - commands used:
     - `[exact command that produced the evidence]`

2. [Finding title]
   - evidence:
     - `path/to/file.rs:789`
   - [explanation]

### Completeness

[Line-by-line comparison of BRIEF.md against the diff. Every item in
the brief must be accounted for.]

- Truthful state: [scaffolded | implemented-but-not-exposed | exposed | release-verified]
- Acceptance criteria: [all addressed | missing — list which]
- Completion matrix: [all applicable cells proven | missing — list which | N/A]
- Implementation approach steps: [all done | missing — list which]
- Done-when conditions: [all met | missing — list which]
- Files listed in brief: [all touched | skipped — list which]
- Architectural intent: [achieved | shortcut — describe if the
  implementation achieves functional correctness through a mechanism
  different from what the brief and plan specified]
- Deferred items: [none | list — for each, state whether deferral is
  backed by a verifiable blocker or is a judgment call. Judgment-call
  deferrals are not accepted.]
- Placeholder audit: [clean | issues — include semantic branch inspection,
  not grep alone]

### Structural Assessment

- Architecture: [clean | issues — describe]
- Abstraction: [clean | issues — describe what should be unified or split]
- Semantic families: [unified | divergence — describe]
- Patterns: [consistent | breaks — describe]
- Integrity: [clean | issues — shortcuts, heuristics, silent failures,
  bug cover-ups, or incomplete error paths found]
- Scope: [clean | out-of-scope changes — describe]

### Forward Impact

- [clean | risks — describe obstacles, wrong interfaces, tight coupling,
  or decisions that may need undoing]
- Small fix now: [none, or describe what would prevent later rework]

### Remediation Mapping

[Map findings to milestones or next steps. Only include if there are
findings that need action.]

- [Milestone or action]:
  - [what needs to happen to resolve finding N]
```

## Notes

- Always include exact `file:line` evidence — the orchestrator and
  implementation agent need precise locations, not vague descriptions.
- Commands used should be exact and re-runnable so findings can be
  verified independently.
- The remediation mapping connects findings to actionable next steps.
  If the orchestrator needs to spawn a fix agent, this section tells
  it what to include in the brief.
- Prior audit sections in the same file provide history. The agent may
  reference them to note progress ("finding 2 from YYYY-MM-DD is now
  resolved") but must not modify them.
