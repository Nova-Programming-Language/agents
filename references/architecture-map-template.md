# Architecture Map Template

Copy into the project as `docs/architecture/file-map.md` (or the location
the project names) and fill in. Rules for using it are in
`architecture-docs.md`.

```md
# Architecture Map

Which architecture doc governs which source paths. Keep it current: a change
that adds, moves, or deletes paths updates the matching rows. Check it with
`[path to the project's map check script, if any]`.

| Paths (globs)        | Governing doc(s)                    | Component |
|----------------------|-------------------------------------|-----------|
| `src/parser/**`      | `frontend.md` | Parser    |
| `src/codegen/**`     | `backend.md`  | Codegen   |
| `src/runtime/io/**`  | `runtime.md`, `io.md` | Runtime I/O |

## Unmapped on purpose

Paths with no governing doc by decision, not by omission (generated code,
vendored code, scripts). Anything not listed here or above is a doc gap.

- `vendor/**` — third-party, not governed
```

## Notes

- Doc names are relative to the directory holding the map.
- The first doc in a row owns the component; later docs also constrain it.
- One row per component boundary, not per file. Prefer directory globs.
- A path may appear in several rows when docs share it; list the owner first.
- The overview doc (e.g. `architecture.md`) is not a governing doc; don't
  map paths to it just to close a gap.
