---
name: remote-tmux-workspace
description: Work in a remote repository or workspace over SSH using durable tmux sessions and clean output capture. Use when the user asks Codex to SSH into another machine, inspect or modify a remote repo, keep a persistent remote shell open, run long remote builds/tests, compare tmux vs screen, or avoid rediscovering SSH/tmux command patterns across sessions.
---

# Remote Tmux Workspace

## Core Practice

Use ordinary non-interactive `ssh` for clean one-shot reads. Use detached `tmux` for durable state, long-running commands, activated environments, dev servers, or workflows that benefit from a persistent remote shell.

Before any networked SSH command, request network permission when the session requires it. Never assume credentials, hostnames, repo paths, or destructive intent beyond what the user gave.

Prefer `tmux` over `screen` when available: it has better scriptable session creation, command injection, resizing, and plain-text pane capture.

## Session Setup

Use a stable session name derived from the task or repo, for example `codex-nova-errmsg`.

Create or reuse a wide detached session rooted at the repo:

```bash
ssh USER@HOST 'tmux has-session -t SESSION 2>/dev/null || tmux new-session -d -s SESSION -x 220 -y 60 -c /path/to/repo'
```

Set remote shell defaults once per session:

```bash
ssh USER@HOST 'tmux send-keys -t SESSION "export NO_COLOR=1 GIT_PAGER=cat CARGO_TERM_COLOR=never" C-m'
```

Check that it exists:

```bash
ssh USER@HOST 'tmux ls | grep SESSION || true'
```

Attach interactively only when needed:

```bash
TERM=xterm-256color ssh -tt USER@HOST 'TERM=xterm-256color tmux attach -t SESSION'
```

If attached output contains terminal control codes, detach and consume output with `capture-pane` instead.

## Clean Command Patterns

For short inspections, prefer direct SSH:

```bash
ssh USER@HOST 'cd /path/to/repo && git -c color.ui=false --no-pager status --short --branch'
ssh USER@HOST 'cd /path/to/repo && git grep -n -I "pattern" -- path/'
```

If `rg` is unavailable remotely, use `git grep` inside Git repos. Keep outputs bounded with `head`, `tail`, or path filters.

For persistent-shell commands, inject into tmux:

```bash
ssh USER@HOST 'tmux send-keys -t SESSION "pwd" C-m'
ssh USER@HOST 'tmux capture-pane -pJ -S -2000 -t SESSION'
```

Use `-pJ` for plain text with joined wrapped lines. Resize wide before large output:

```bash
ssh USER@HOST 'tmux resize-window -t SESSION -x 220 -y 60'
```

## Long Commands

For builds/tests, write logs on the remote side and tail them through clean SSH:

```bash
ssh USER@HOST 'tmux send-keys -t SESSION "cd /path/to/repo && cargo test > /tmp/codex-test.log 2>&1; printf \"\\n__CODEX_DONE:\$?__\\n\"" C-m'
ssh USER@HOST 'tail -200 /tmp/codex-test.log'
```

Poll with `capture-pane` only for completion/sentinel state; use `tail` or `cat` for the actual evidence.

## Repository Hygiene

On first contact with a remote repo:

```bash
ssh USER@HOST 'cd /path/to/repo && git -c color.ui=false --no-pager status --short --branch'
ssh USER@HOST 'cd /path/to/repo && sed -n "1,200p" AGENTS.md 2>/dev/null || true'
```

Respect existing dirty changes as user-owned. Before editing, read repo instructions such as `AGENTS.md` and any files it names. For edits, prefer patches or normal repo workflows; verify with `git diff` and targeted tests. Do not use destructive git commands unless the user explicitly asks.

## Editing Over SSH

For remote file edits, prefer a patch-oriented flow:

```bash
ssh USER@HOST 'cd /path/to/repo && git apply --check -' < /path/to/local.patch
ssh USER@HOST 'cd /path/to/repo && git apply -' < /path/to/local.patch
```

For small, task-local generated files, a heredoc may be acceptable, but avoid ad hoc rewrites of existing source when a patch is practical.

## Closeout

Report:

- remote host and repo path
- tmux session name and whether it remains attached/detached
- branch/dirty status before edits
- files changed, if any
- verification run and result

Leave durable sessions running only when useful, and tell the user the session name.
