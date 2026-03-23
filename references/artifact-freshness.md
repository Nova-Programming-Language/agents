# Artifact Freshness

Use this reference when deciding whether a Nova validation command can trust the
artifacts it is about to execute.

## Core Rule

Never assume existing Nova build artifacts are fresh after source edits.

Before running validation, answer three questions:

1. What command am I about to run?
2. Which binary or libraries does that command actually execute?
3. Does the command rebuild those artifacts itself, or do I need to refresh them first?

## Artifact Matrix

| Validation surface | Example command | Rust CLI freshness | Runtime library freshness | Required action |
| --- | --- | --- | --- | --- |
| Cargo compile-only | `cargo check -p nova-cli` | handled by Cargo | not relevant | none beyond the command itself |
| Cargo debug execution | `cargo run -p nova-cli -- test tests/unit/ --skip-compiled` | handled by Cargo for the debug CLI | not guaranteed | no manual CLI rebuild; rebuild runtimes only if the path exercises compiled/runtime-linked behavior |
| Cargo crate tests | `cargo test -p nova-cli <filter>` | handled by Cargo for debug test binaries | not guaranteed | no manual CLI rebuild; rebuild runtimes only if the test depends on them |
| Release CLI execution | `target/release/nova test ...` | not handled automatically | not guaranteed | run `cargo build --release -p nova-cli` first |
| Compiled/runtime-linked validation | `cargo run -p nova-cli -- test tests/unit/ --compiled` | handled by Cargo for the debug CLI | not handled automatically | rebuild `nova-async-rt` and `runtime` before trusting the result |
| Release compiled validation | `target/release/nova test --compiled ...` | not handled automatically | not handled automatically | rebuild release CLI plus both runtime libraries first |
| Canonical full regression | `scripts/full-regression.sh check` | handled by the script | handled by the script | no extra manual rebuild needed |
| x86 container validation | `scripts/nova-x86-container.sh regression check` | depends on container state | depends on container state | run `scripts/nova-x86-container.sh prepare` before trusting stale container artifacts |

## Required Refresh Commands

Release CLI:

```bash
cargo build --release -p nova-cli
```

Runtime artifacts used by compiled/runtime-linked paths:

```bash
cargo build --release -p nova-async-rt
make -C runtime
```

Pinned Linux/x86_64 container reset:

```bash
scripts/nova-x86-container.sh prepare
```

## Practical Rules

- `cargo run -p nova-cli -- ...` keeps the Rust CLI fresh for that command, but it does not guarantee that `nova-async-rt` or `runtime/libnova_runtime.a` are fresh.
- `target/release/nova ...` is always suspicious after source edits unless you just rebuilt it.
- If compiled backend behavior, runtime ABI, or native linkage is in scope, refresh both runtime artifact surfaces before trusting the result.
- If you want the safest full-suite path, use `scripts/full-regression.sh` instead of hand-assembling release and runtime rebuilds.
- When reporting validation, state what was rebuilt or why the command itself guaranteed freshness.
