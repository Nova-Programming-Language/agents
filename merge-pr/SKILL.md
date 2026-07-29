---
name: merge-pr
description: Use this skill when the user asks to merge, land, close out, or verify the merge of a GitHub pull request in the Nova repo. This skill uses the gh CLI to confirm the real PR object, verify mergeability and state, merge with the intended strategy, and then verify that the PR is actually merged and that origin/main moved.
---

# Merge PR

Use this skill for merging GitHub pull requests in the Nova repository.

This skill is Nova-specific where it matters:

- use `gh` for PR operations
- treat `main` as the default merge target unless the user says otherwise
- prefer a normal merge commit for Nova unless the user explicitly asks for
  squash or rebase
- verify the actual merged PR state and the updated remote branch before
  reporting success
- do not trample dirty local worktrees while syncing local branches after merge

## When to Use

Use this skill when the task is specifically to:

- merge a PR
- verify whether a PR was actually merged
- determine why a PR cannot be merged yet
- sync local `main` after a PR merge

Do not use this skill for PR creation. Use the companion `create-pr` skill for
that.

## Required Rules

- Use `gh` for all GitHub PR merge operations.
- Do not claim a PR is merged until `gh pr view` shows `state: MERGED`.
- Do not assume a successful command means local `main` is current; verify
  `origin/main`.
- Do not switch or rebase a dirty local branch without checking whether that
  would preserve local worktree changes.

## Workflow

### 1. Identify the PR

If the user gave a PR number:

```bash
gh pr view <number> --json number,title,url,state,isDraft,mergeStateStatus,headRefName,baseRefName
```

If the user referred to a branch:

```bash
gh pr list --state open --head <branch> --json number,title,url,state,isDraft,mergeStateStatus,headRefName,baseRefName
```

Confirm:

- PR number
- PR URL
- head branch
- base branch
- draft vs ready
- current state

If no PR exists, stop and say so plainly.

### 2. Check merge readiness

Inspect the PR before merging:

```bash
gh pr view <number> --json number,title,url,state,isDraft,mergeStateStatus,reviewDecision,statusCheckRollup,headRefName,baseRefName
```

At minimum verify:

- PR is open
- PR is not draft unless the user explicitly wants to override that
- merge state is acceptable
- required checks are not obviously failing or pending

If merge is blocked, report the blocker instead of forcing the merge.

### 3. Merge the PR

For Nova, default to a normal merge commit unless the user explicitly asks for
another strategy.

Typical form:

```bash
gh pr merge <number> --merge
```

Only use:

- `--squash` when the user explicitly wants a squash merge
- `--rebase` when the user explicitly wants a rebase merge

If branch deletion is desired, request it explicitly through the `gh` flags
used at merge time rather than assuming.

### 4. Verify the merge actually happened

After merge, verify:

```bash
gh pr view <number> --json number,state,mergedAt,mergeCommit,url,headRefName,baseRefName
```

Do not report success until all of these are present:

- `state` is `MERGED`
- `mergedAt` exists
- `mergeCommit` exists

### 5. Sync local state carefully

If the user wants local `main` synced after merge:

1. Inspect the local worktree first.

```bash
git status --short
git branch --show-current
git fetch origin
git rev-list --left-right --count HEAD...origin/main
```

2. If the worktree is clean, fast-path sync:

```bash
git switch main
git pull --rebase origin main
```

3. If the worktree is dirty:

- do not blindly switch branches or rebase
- preserve local changes first
- explain exactly what is blocking the sync

## Nova-Specific Defaults

- Base branch is normally `main`.
- Merge strategy is normally `--merge`.
- A merged PR should usually be followed by a fetch and branch-divergence check
  against `origin/main`.
- If local worktree changes exist, especially generated files, do not hide them
  during post-merge sync.

## Failure Handling

If `gh pr merge` appears to succeed silently, immediately verify with
`gh pr view`. If the PR is still open, it was not merged.

If the command is still running, poll it and then re-check the PR state.

If the merge failed because the PR became stale, fetch and re-check divergence
before deciding on the next step.

## Anti-Patterns

Do not:

- report a PR as merged based only on command exit status
- assume local `main` moved just because GitHub merged the PR
- force a merge strategy the user did not ask for
- switch/rebase through a dirty local worktree without preserving it

## Minimal Output Standard

A correct merge result looks like:

- `Merged PR #123: https://github.com/org/repo/pull/123`
- `Merge commit: abcdef123`
- `Local main is now X behind origin/main`

or, if blocked:

- `PR #123 is not mergeable yet: <reason>`
