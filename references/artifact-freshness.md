# Artifact Freshness

Use this reference when deciding whether a Nova validation command can trust the
artifacts it is about to execute.

## Core Rule

Never assume existing Nova build artifacts are fresh after source edits.

Preferred default:

- use the release CLI, `cargo run --release -q -p nova-cli -- ...`, for normal
  Nova CLI work
- use the debug CLI, `cargo run -q -p nova-cli -- ...`, only when you
  intentionally want it

Name the build profile, not a path. Cargo resolves the target directory, so
these keep working where build artifacts are redirected; `target/release/nova`
does not exist in every checkout.

Before running validation, answer three questions:

1. What command am I about to run?
2. Which binary or libraries does that command actually execute?
3. Does the command rebuild those artifacts itself, or do I need to refresh them first?

## Artifact Matrix

| Validation surface | Example command | Rust CLI freshness | Runtime library freshness | Required action |
| --- | --- | --- | --- | --- |
| Cargo compile-only | `cargo check -p nova-cli` | handled by Cargo | not relevant | none beyond the command itself |
| Debug CLI execution | `cargo run -q -p nova-cli -- test tests/unit/ --backend interpreter` | handled by Cargo for the debug CLI | not guaranteed | debug-only; prefer the release CLI unless you are intentionally validating the debug binary |
| Cargo crate tests | `cargo test -p nova-cli <filter>` | handled by Cargo for debug test binaries | not guaranteed | no manual CLI rebuild; rebuild runtimes only if the test depends on them |
| Release CLI execution | `cargo run --release -q -p nova-cli -- test ...` | handled by Cargo for the release CLI | not guaranteed | the CLI is built as part of the command; runtime surfaces are not |
| Compiled/runtime-linked validation | `cargo run -q -p nova-cli -- test tests/unit/ --backend c` | handled by Cargo for the debug CLI | not handled automatically | debug-only; prefer the release CLI; rebuild `nova-async-rt` and `runtime` before trusting the result |
| Release compiled validation | `cargo run --release -q -p nova-cli -- test --backend c ...` | handled by Cargo for the release CLI | not handled automatically | rebuild both runtime libraries first |
| Canonical full regression | `scripts/run-nova-full-regression.sh check` | handled by the script | handled by the script | no extra manual rebuild needed |
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

- For normal repo or package work, invoke Nova through the release CLI:
  `cargo run --release -q -p nova-cli -- ...`.
- Running the CLI through Cargo builds it as part of the command, so the CLI
  itself cannot be stale. It does not guarantee that `nova-async-rt` or
  `runtime/libnova_runtime.a` are fresh.
- The debug CLI builds a separate set of large debug artifacts, so do not use
  it as the routine path.
- A binary invoked by path is always suspicious after source edits unless you
  just rebuilt it. Resolve one with `cargo metadata --format-version 1
  --no-deps` (`target_directory`), or set `NOVA_BIN`, which the repository
  scripts honor.
- If compiled backend behavior, runtime ABI, or native linkage is in scope, refresh both runtime artifact surfaces before trusting the result.
- If you want the safest full-suite path, use `scripts/run-nova-full-regression.sh` instead of hand-assembling release and runtime rebuilds.
- When reporting validation, state what was rebuilt or why the command itself guaranteed freshness.
