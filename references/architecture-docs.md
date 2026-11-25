# Architecture Docs: Finding, Gaps, and Currency

Use this reference whenever a skill reads architecture docs to decide where
code belongs. It is the one home for which doc governs a change, what to do
when none does, and when a change must update the docs.

## The architecture map

Each project keeps one map from source paths to the architecture docs that
govern them. Default location: `docs/architecture/file-map.md`. A project may
name a different file, or give the map itself as a path → doc list, in profile
§Paths (`project-profile.md`), PROJECT.md, AGENTS.md, or CLAUDE.md; use that if
present.

Format and starting point: `architecture-map-template.md` (next to this file).

To find the governing doc for each changed file, match its path against the
map's globs. A path matched by several rows is governed by every doc listed.
Record the docs used. For a changed path no row matches, say how a doc was
chosen instead (content match, the overview doc) — or that none applies.

If the project has no map at all, that is a doc gap for every changed path.

If the map names a check script, run it after a change that adds, moves, or
deletes source paths.

## When no doc governs the change

A changed path with no map row, or whose governing doc lacks an
**Invariant:**, a **Not responsible for:**, or a Common Change Pattern for
this kind of change, is a **doc gap**. A doc gap is never a silent skip:

- **Implementing or fixing**: follow the structure of the surrounding code,
  do not invent new boundaries, and record the gap in STATUS.md (or the
  diagnosis).
- **Audit**: report the Architecture check as `unverified — doc gap`, list
  each gap as a finding, and map its remediation to `design`. Unverified is
  not clean.

## When a change must update the docs

A change must update the governing doc or the map in the same change when it:

- moves ownership of a behavior between components;
- adds, removes, or weakens a boundary or an **Invariant:**;
- adds a kind of change that has no matching Common Change Pattern; or
- adds, moves, or deletes source paths so that the map no longer matches
  (add, update, or remove the row).

Small updates (a map row, a new Common Change Pattern entry, a corrected
invariant) are part of the change itself, written to the `design` skill's
format. If the update would redraw a boundary or restructure a doc, stop and
route it to `design` before implementing — the code must not get ahead of the
architecture.

The audit treats a missing required update as a structural finding.
