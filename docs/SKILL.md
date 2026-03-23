---
name: docs
description: Document Nova code and use nova doc correctly; use when writing doc comments, validating examples, or generating API docs for the Nova repo.
---

# Docs

## When to Use

Use this skill when editing Nova doc comments, verifying documentation examples,
or running `nova doc`.

## Source of Truth

Read these first:

1. `docs/specs/lang/tooling.md`
2. `docs/nova-cli.md`
3. `../references/artifact-freshness.md` when validating local `nova doc`
   behavior after CLI changes

The documentation-comments section in `tooling.md` is the semantic source of
truth. `nova-cli.md` is the command reference for `nova doc`.

## Documentation Comment Rules

Documentation comments are ordinary `#` comments immediately before a
declaration.

Supported sections include:

- first paragraph summary
- `Args:`
- `Returns:`
- `Errors:`
- `Example:`
- `Syntax:`
- `See also:`

## `Example:` vs `Syntax:`

- `Example:` is executable usage and is valid only in `.nova`.
- `Syntax:` is parse/type-check-only and is preferred for `.novi`.
- `.novi` must not use executable `Example:` snippets.

When unsure:

- use `Example:` for runnable API usage in implementation modules
- use `Syntax:` for interfaces and non-executable signatures

## Verification Commands

Shape/signature validation:

```bash
nova doc --check
nova doc --check --strict
```

Generation:

```bash
nova doc
nova doc -f markdown
nova doc -f html -o docs/api-html
```

`--strict` is the right path when public items must fail on missing docs.

## Example Expectations

`nova doc` verifies snippets differently:

- `Example:` snippets in `.nova` parse, type-check, and execute
- `Syntax:` snippets parse and type-check only

This means doc changes can fail because the examples are stale even when the
main code still builds.

## Operational Workflow

1. Update doc comments alongside API changes.
2. Check whether the item is public and whether `--strict` should pass.
3. If validating local CLI behavior, decide whether you are using `cargo run`
   or a release binary and apply `../references/artifact-freshness.md`.
4. Run `nova doc --check` on the affected path.
5. If examples changed, verify the examples too.
6. Generate final docs only after the checks are clean.
