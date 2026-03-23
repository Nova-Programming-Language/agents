# When To Write A PRD

Use this file to decide whether a project needs a real PRD.

## Write A Real PRD When

- the project introduces a new user-visible capability
- the project changes externally visible behavior in a way that needs stable
  intent across sessions
- the work spans multiple components and requirements are likely to drift
- the acceptance criteria cannot be derived cleanly from existing specs and
  architecture alone
- future sessions would otherwise need to infer “what this project is trying to
  achieve” from chats or commits

## Use `PRD: N/A` When

- the task is a narrow bug fix already scoped by an existing spec or
  architecture document
- the work is purely internal cleanup with no product-level behavior change
- the project is so small that `PROJECT.md` can hold the requirements without
  losing clarity

## PRD Boundaries

A PRD should answer:

- what problem is being solved
- for whom
- what outcomes are intended
- what is in scope and out of scope
- what acceptance criteria define success

A PRD should not become:

- an architecture doc
- an implementation plan
- a session log
- a design-by-code-inventory document
