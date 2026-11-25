---
name: merge-pr
description: Merge a GitHub pull request with gh and verify it — confirm the PR and its mergeability, merge with the intended strategy, then check it is MERGED and the remote default branch moved; use when asked to merge, land, close out, or verify a merge. Not for creating PRs (use create-pr).
---

# Merge PR

Use this skill for merging GitHub pull requests in the project repository.

Project-specific facts come from profile §Repository (see
`../references/project-profile.md`): the remote, the default branch (written
`<default>` below), and the default merge strategy. Throughout:

- use `gh` for PR operations
- treat `<default>` as the merge target unless the user says otherwise
- use the profile's merge strategy unless the user explicitly asks for another;
  if the profile names none, ask
- verify the actual merged PR state and the updated remote branch before
  reporting success
- do not trample dirty local worktrees while syncing local branches after merge

## When to Use

Use this skill when the task is specifically to:

- merge a PR
- verify whether a PR was actually merged
- determine why a PR cannot be merged yet
- sync local `<default>` after a PR merge

Do not use this skill for PR creation. Use the companion `create-pr` skill for
that.

## Required Rules

- Use `gh` for all GitHub PR merge operations.
- Do not claim a PR is merged until `gh pr view` shows `state: MERGED`.
- Do not assume a successful command means local `<default>` is current;
  verify `origin/<default>`.
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

Use the profile's merge strategy unless the user explicitly asks for another.

Typical form:

```bash
gh pr merge <number> --merge|--squash|--rebase
```

Use a strategy other than the profile's only when the user explicitly asks
for it.

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

If the user wants local `<default>` synced after merge:

1. Inspect the local worktree first.

```bash
git status --short
git branch --show-current
git fetch origin
git rev-list --left-right --count HEAD...origin/<default>
```

2. If the worktree is clean, fast-path sync:

```bash
git switch <default>
git pull --rebase origin <default>
```

3. If the worktree is dirty:

- do not blindly switch branches or rebase
- preserve local changes first
- explain exactly what is blocking the sync

## Defaults

- Base branch is the profile's default branch.
- Merge strategy is the profile's; ask if it names none.
- A merged PR should usually be followed by a fetch and branch-divergence check
  against `origin/<default>`.
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
- assume local `<default>` moved just because GitHub merged the PR
- force a merge strategy the user did not ask for
- switch/rebase through a dirty local worktree without preserving it

## Minimal Output Standard

A correct merge result looks like:

- `Merged PR #123: https://github.com/org/repo/pull/123`
- `Merge commit: abcdef123`
- `Local <default> is now X behind origin/<default>`

or, if blocked:

- `PR #123 is not mergeable yet: <reason>`
