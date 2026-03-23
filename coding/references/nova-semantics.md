# Nova Semantic Reminders

Load this file when the implementation touches core Nova language semantics.

- `mut` and `const` are orthogonal.
- `Option!T = T | None`.
- `Result!T = T | Error`.
- There is no Rust-style `Some(...)`.
- `?` and `!` are the only special propagation/unwrap operators.
- `prelude.novi` is the runtime contract; do not drift from it casually.
- The frontend is an atomic semantic unit. Do not split parsing/type-checking
  assumptions across ad hoc consumers.
