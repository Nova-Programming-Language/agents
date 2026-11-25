---
name: test-authoring
description: Write or update project tests with the correct form, annotations, and validation flow; use when adding unit, program, or diagnostic coverage. Not for running regression suites (use regressions) or fixing failing tests (use fix-test).
---

# Test Authoring

## When to Use

Use this skill when a change needs new tests, updated annotations, or a
decision about where a test belongs in the project's test layout.

Test layout, kinds, file conventions, annotations, and run commands come from
profile §Tests; find the profile per `../references/project-profile.md`.

## Source of Truth

Read these in order before choosing a test strategy:

1. the project's test spec, test-system architecture doc, and test guide
   (profile §Tests)
2. `../references/artifact-freshness.md` when compiled or runtime-linked
   validation is in scope

The spec defines language-level test semantics. The architecture and guide
define how the test runner classifies files, records cases, and treats
annotations.

## Choose the Right Test Surface

Use the smallest test surface that proves the behavior. Profile §Tests lists
the test kinds (for example diagnostic, unit, program, end-to-end examples),
their typical placement, and any path-based tier defaults and overrides.
End-to-end examples are for public workflows, not a substitute for focused
unit or diagnostic coverage.

Directory placement is a convention, not the semantic source of truth: test
form and engine come from the file contents plus annotation semantics.

## Required Annotation Rules

Follow the annotation rules in profile §Tests: which annotations apply to
which test form, and whether each applies to the next test or the whole file.

Common authoring mistakes:

- putting a per-test annotation (such as a skip) on a helper and expecting it
  to affect another test
- using a diagnostic-only annotation in a runtime test instead of a
  diagnostic file
- storing blocked work in a special suite instead of using explicit skip or
  pending annotations
- relying on directory name alone when the file content actually changes form

## Authoring Checklist

1. Add or update success-path coverage.
2. Add or update invalid-usage or rejected-syntax coverage when relevant.
3. Cover edge and failure behavior, not only the happy path.
4. Keep skipped tests parseable and type-checkable; skip is not a syntax
   escape hatch.
5. Know how the runner records cases for the file's form (profile §Tests).
6. Fix new warnings instead of leaving them for later, per
   `../coding/SKILL.md`, "Implementation Rules".

## Validation Workflow

Run the narrowest command that proves the new test (profile §Tests: how to
run one test or one file).

A passing run is not proof that anything ran: check the executed count per
`../regressions/SKILL.md`, "Non-Negotiable Rules".

Use differential validation across execution engines or backends when the
change can diverge between them, using the run modes profile §Tests
describes. Read engine outcomes from the structured report fields, not from
free-text messages.

Do not trust compiled results after source edits unless the runtime-linked
artifacts were refreshed first. Use `../references/artifact-freshness.md` and
profile §Build to decide which artifacts need a rebuild.

For broader confirmation, hand off to the regression workflow skill and use
the canonical suite entrypoint from profile §Regression. Do not describe a
scoped test run over a directory as canonical full regression.

When the test runner's ordinary output could hide test-only compiler
warnings, use the warning-deny compile check from profile §Build for the
affected component.
