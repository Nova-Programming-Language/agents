---
name: create-pr
description: Use this skill when the user asks to create, open, submit, verify, or check a GitHub pull request in the Nova repo. This skill uses the gh CLI to avoid duplicate PRs, ensures the branch is pushed, creates the PR only when no PR already exists for the head branch, and verifies the actual PR number, URL, and state before reporting success.
---

# Create PR

Use this skill for GitHub pull request creation and verification in the Nova
repository.

This is a narrow workflow skill. It is not for general git commits or generic
GitHub browsing. Use it when the task is specifically to:

- create or open a PR
- verify whether a PR already exists for a branch
- get the real PR URL/number/state
- avoid duplicate PRs for the same head branch

This skill is Nova-specific where it matters:

- use `gh` for all PR operations
- treat `main` as the default base branch unless the user says otherwise
- push the working branch before creation
- verify the actual PR object after creation

## Required Rules

- Use `gh` for all GitHub PR operations.
- Do not treat a prefilled `/pull/new/...` URL as a created PR.
- Do not claim success until `gh` returns a real PR object and you verify it.
- Prefer reusing an existing PR for the same head branch instead of creating a duplicate.

## Workflow

### 1. Inspect local and remote state

Run:

```bash
git fetch origin
git branch --show-current
git status --short
git rev-list --left-right --count HEAD...origin/main
```

Also inspect the target branch if it is not the current branch.

### 2. Check whether a PR already exists

Use the actual head branch name:

```bash
gh pr list --state open --head <branch> --json number,title,url,state,isDraft,headRefName,baseRefName
gh pr list --state all --head <branch> --json number,title,url,state,isDraft,headRefName,baseRefName
```

If an open PR already exists:

- do not create another one
- report the existing PR number and URL
- push new commits to the same branch if the user wants the PR updated

If a closed or merged PR exists for the same branch, inspect whether the branch
was reused intentionally before creating a new PR.

### 3. Ensure the head branch is pushed

If the remote branch does not exist yet or local is ahead:

```bash
git push -u origin <branch>
```

If upstream moved and push fails, sync first using the repo’s normal rebase
policy before retrying.

### 4. Create the PR

Use `gh pr create` with explicit `--base` and `--head`.

For Nova, default the base to `main` unless the user explicitly asks for a
different target branch.

Typical form:

```bash
gh pr create \
  --base main \
  --head <branch> \
  --title "<title>" \
  --body "<body>"
```

Use `--draft` only when the user asks for a draft or the work is explicitly not
ready.

Write the PR title/body from the actual branch contents. Keep them factual and
software-focused. In Nova, the PR body should summarize the actual software
changes and validation, not agent process notes.

### 5. Verify the PR object exists

Immediately verify after `gh pr create`:

```bash
gh pr list --state open --head <branch> --json number,title,url,state,isDraft
```

or:

```bash
gh pr view <number> --json number,title,url,state,isDraft,headRefName,baseRefName
```

Do not stop at command exit status alone. Confirm:

- PR number
- PR URL
- state
- head branch
- base branch

### 6. Report the result precisely

When successful, report:

- whether the PR was newly created or already existed
- PR number
- PR URL
- head branch
- base branch
- whether it is draft or ready for review

## Failure Handling

If `gh pr create` behaves unexpectedly:

- poll the command if it is still running
- check whether it opened an interactive flow or browser-backed creation path
- re-run the `gh pr list --head <branch>` verification step

If no PR object exists afterward, creation did not complete. Do not report a
PR URL unless it comes from a real `gh` PR object.

## Nova-Specific Defaults

- Base branch is normally `main`.
- Prefer one PR per working branch.
- If the branch already has an open PR, update that PR rather than creating a
  second one.
- A pushed branch is not enough; the PR must be verified through `gh`.

## Anti-Patterns

Do not:

- report `https://github.com/<org>/<repo>/pull/new/<branch>` as if it were a PR
- assume a pushed branch means a PR exists
- create a second PR without checking for an existing one on the same head branch
- use ad hoc web scraping instead of `gh`

## Minimal Output Standard

A correct PR-creation result looks like:

- `Created PR #123: https://github.com/org/repo/pull/123`

or:

- `PR already exists: #123 https://github.com/org/repo/pull/123`

Anything weaker is incomplete.
