---
name: bug-report
description: Write verified bug reports with exact reproduction, clean-rebuild status, and ownership clues; use when documenting a reproducible defect or blocker.
---

# Bug Report

## When to Use

Use this skill for local investigation of a bug — reproducing it, narrowing
the subsystem, and writing durable notes. The output lives in the reports
directory named in profile §Paths.

Project-specific facts (report location, components, default branch)
come from the project profile; see `../references/project-profile.md` for how
to find it.

To track a bug externally in the issue tracker (profile §Repository), use the
`issues` skill instead.
Use `bug-report` first when the reproduction is non-trivial and you
need structured investigation before filing.

## Requirements Before Writing

Do not write the report from memory alone.

Confirm:

1. the exact command or file that reproduces the issue
2. the current branch or commit under test
3. whether the issue still reproduces after a clean rebuild when relevant
4. which project component the bug is on (use the architecture map, profile
   §Paths)

If the bug no longer reproduces, the report should say so plainly and record
what changed during verification.

## Standard Report Structure

Use the template in `references/template.md`.

Each report should include:

- `Summary`
- `Reproduction`
- `Expected`
- `Actual`
- `Scope`
- `Ownership Hypothesis`
- `Verification Notes`
- `Next Action`

Keep the write-up factual. Include exact commands, file paths, and observed
errors. Avoid speculative prose that is not tied to evidence.

## Storage Rules

Store repo-local reports in the reports directory from profile §Paths, using
the file naming pattern it gives (typically a date plus a short slug).

Use the current local date, not a guessed date.

## Writing Rules

- Say whether the reproducer was run on the current default branch (profile
  §Repository), a feature branch, or a dirty worktree.
- Distinguish “originally observed” from “currently reproducible”.
- If a clean rebuild changes the outcome, state that explicitly.
- If the issue blocks another task, say which commands or workflows are blocked.
- If ownership is uncertain, narrow it to the most likely subsystem instead of
  naming a person.

## Verification Workflow

Typical workflow:

1. reproduce the issue with the narrowest command
2. rerun with any necessary clean rebuild
3. capture the exact failing output or behavior
4. identify the most likely subsystem
5. write the report in the reports directory (profile §Paths)

This skill is for durable reports, not transient debug notes.
