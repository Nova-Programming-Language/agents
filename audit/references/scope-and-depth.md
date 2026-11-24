# Scope and Depth

How an audit decides what changed and how deep to check it.

## Base and diff

Find the base in this order, and record which source it came from:

1. A base commit supplied by the caller (the orchestrator passes one).
2. The base recorded in the project's STATUS.md, BRIEF.md, or PLAN.md.
3. For a single uncommitted fix (the `fix-test` flow): `HEAD`.
4. Otherwise `git merge-base HEAD @{upstream}`. If that fails, stop and ask
   for a base — do not guess.

Audit the working tree, not just commits — work is usually audited before it
is committed:

- `git diff <base>` — committed, staged, and unstaged changes to tracked files
- `git status --porcelain` — untracked files, which `git diff` omits; read
  every new file that belongs to the change

## Depth

Pick a depth, record it and the reason, and run only that depth's checks.
Depth never skips an audit: small changes are where shortcuts hide.

**Focused**
- Use when: one component, no public contract or interface change, not a
  refactor or migration (typically a single test fix).
- Run: functional criteria, Architecture, Repairs, Integrity, Scope;
  Patterns and Function
  Size on touched lines only; forward impact in one line.
- Skip: completion matrix, Refactoring Integrity, the Abstraction survey of
  neighboring files.

**Full** — everything else, including a milestone gate or any focused check
that finds a structural problem. Run every check.

When unsure, use full.
