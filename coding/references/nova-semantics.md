# Nova Semantic Reminders

Load this file when the implementation touches core Nova language semantics.

- `mut` and `const` are orthogonal.
- `Option!T` / `Result!T` semantics live in
  `docs/specs/lang/prelude.md#core-composite-types` and
  `docs/specs/lang/errors.md`.
- Binding-family semantics live in
  `docs/specs/lang/syntax.md#binding-introducing-constructs`,
  `docs/specs/lang/memory.md#ownership-annotations`, and
  `docs/architecture/semantic-families.md`.
- There is no Rust-style `Some(...)`.
- `?` and `!` are the only special propagation/unwrap operators, and they are
  the explicit divergence points for `Option!T` / `Result!T` rather than a
  reason to treat those types as bespoke everywhere else.
- `prelude.novi` is the runtime contract; do not drift from it casually.
- The frontend is an atomic semantic unit. Do not split parsing/type-checking
  assumptions across ad hoc consumers.
