---
name: prd
description: "Write or update Nova project PRDs for multi-session work; use when a project needs an explicit product-level problem statement, scope, user outcomes, and acceptance criteria instead of `PRD: N/A`."
---

# PRD

## When to Use

Use this skill when a Nova project needs a real PRD rather than a `PRD: N/A`
entry in `PROJECT.md`.

Typical triggers:

- new user-visible capability
- broad behavior change across multiple components
- feature or refactor work that spans multiple commits and needs stable product
  intent across sessions
- requirements are drifting because the project lacks a durable product-level
  problem statement

Do not use this skill for narrow bug fixes already fully scoped by existing
specs and architecture. In that case, keep `PRD: N/A` with a reason.

## Artifact

Store PRDs with the delivery artifacts:

```text
project-notes/<slug>/PRD.md
```

`PROJECT.md` should link to that file in its required links section.

## Read First

- `docs/agent-rules.md`
- `../references/source-of-truth.md`
- `../references/semantic-family.md` when the project spans related constructs
- `../references/nova-delivery-checks.md`
- `references/when-to-write-a-prd.md`
- `references/prd-template.md`

## Non-Negotiable Rules

- A PRD defines the problem, intended outcomes, scope, and acceptance criteria.
- A PRD is not an architecture document and not an implementation plan.
- Keep product intent stable even if implementation details change later.
- Acceptance criteria must be concrete enough for `PROJECT.md`, `PLAN.md`, and
  validation work to consume directly.
- A Nova PRD must name the user-visible Nova surfaces or workflows affected.
- If a PRD is not needed, say `N/A` with a reason rather than creating a fake
  PRD.

## Workflow

1. Decide whether the project needs a real PRD or `PRD: N/A` by using
   `references/when-to-write-a-prd.md`.
2. If a PRD is needed, create or update `project-notes/<slug>/PRD.md` using
   `references/prd-template.md`.
3. Keep the PRD focused on user problem, outcomes, scope, and acceptance
   criteria, not architecture or code structure.
4. Use `../references/nova-delivery-checks.md` to name the affected Nova
   surfaces and observable behavior clearly.
5. Update `PROJECT.md` so its `PRD` link points to the PRD file.
6. Ensure the acceptance criteria in `PRD.md` and `PROJECT.md` agree.

## Open These References As Needed

- `../references/nova-delivery-checks.md`
- `references/when-to-write-a-prd.md`
- `references/prd-template.md`
