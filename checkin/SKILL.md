---
name: checkin
description: Create commits, rebase onto upstream, and push Nova repo changes; use when the user asks to commit, sync, or publish work.
---

# Checkin

## When to Use

Use this skill when the task is to commit changes, prepare a branch for push,
or publish work to the remote repository.

## Workflow

1. Fetch the remote before proposing a commit or push.
2. Inspect divergence from upstream.
3. Inspect dirty files and draft a commit message from the actual worktree.
4. Wait for the human to confirm the commit message before committing.
5. Stage all dirty files by default.
6. Only split or exclude files when the human explicitly asks for a narrower commit.
7. Commit with a non-interactive git command.
8. Push. If push is rejected because upstream moved, fetch/rebase/push again.

## Required Inspection Commands

```bash
git fetch origin
git branch --show-current
git rev-list --left-right --count HEAD...origin/main
git log --oneline --decorate --left-right HEAD...origin/main -n 10
git status --short
git diff --stat
```

## Rebase Policy

- If local is behind upstream, recommend rebasing onto `origin/main` before
  push.
- Prefer:

```bash
git pull --rebase origin main
```

- Do not create a merge commit unless the user explicitly wants that.
- After a rebase, rerun the appropriate build or validation command if the
  rebase changed code you rely on.

## Commit Policy

- Never create a commit whose only content is session bookkeeping —
  STATUS.md checkpoints, project-notes/audit records, gate-result notes.
  Commits must carry product value external users care about: code,
  tests, or user-facing documentation. Keep bookkeeping edits in the
  working tree and fold them into the next product commit.
- Draft the message from the actual dirty worktree, not from the ticket title.
- Commit messages describe **what changed in the software and why** — not
  the agent process that produced the change. Do not include: validation
  commands run, STATUS.md updates, artifact refreshes, audit results,
  skill invocations, or session workflow steps. Those belong in LOG.md
  or STATUS.md, not in permanent git history.
- Write commit messages as durable context for both humans and future AI coding
  agents. Prefer messages that capture the concrete behavior change, important
  semantic or ownership clues, key validation commands, and any still-relevant
  blockers when that context will help a later coding session.
- Reference GitHub issues in the commit message:
  - If the commit fully fixes an issue: `Fixes #123` (GitHub auto-closes it)
  - If the commit partially addresses an issue: `Progress on #123` or
    `Addresses #123 — [what this commit does toward the fix]`
  - If the commit is related but does not fix: `Related to #123`
  - Check the issue tracker before committing — if the work was driven by
    an issue, the commit must reference it. Do not leave the connection
    implicit.
- Wait for explicit human confirmation before running `git commit`.
- By default, treat the full dirty worktree as the intended commit scope.
- Do not silently narrow the commit to only the files touched most recently.
- Only create a partial commit when the human explicitly asks to split the work
  or exclude files.
- Do not amend an existing commit unless the user asks.
- Do not use destructive git commands such as `reset --hard` or checkout-based
  reverts unless explicitly requested.

## Push Policy

- Preferred push path:

```bash
git push origin <branch>
```

- If rejected as non-fast-forward:
  1. `git fetch origin`
  2. inspect divergence
  3. `git pull --rebase origin <branch>`
  4. push again

## Post-Commit Validation

Use repo judgment:

- small CLI/interpreter change: rerun targeted `cargo check` or targeted tests
- broader change: suggest or run the relevant regression workflow

For full-suite regression, prefer:

```bash
scripts/run-nova-full-regression.sh check
```

## Communication Expectations

- Tell the user when upstream drift blocks push.
- Tell the user exactly which commit was created.
- Tell the user whether the branch was rebased before push.
