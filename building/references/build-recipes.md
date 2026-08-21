# Build Recipes

Use this file after the SKILL tells you which build surface you need.

## Fast Rust Validation

```bash
cargo check -p nova-cli
cargo check -p nova-interpreter -p nova-cli -p nova-test-runner
```

## Debug Builds

```bash
cargo build -p nova-cli
cargo test -p nova-cli
```

These keep debug Rust artifacts fresh. They do not refresh the release CLI or
the runtime libraries used by compiled/runtime-linked paths.

## Release Surfaces

Release CLI:

```bash
cargo build --release -p nova-cli
```

Preferred Nova CLI invocation after that build:

```bash
cargo run --release -q -p nova-cli -- check ...
cargo run --release -q -p nova-cli -- test ...
cargo run --release -q -p nova-cli -- build ...
```

Runtime artifacts used by compiled/runtime-linked paths:

```bash
cargo build --release -p nova-async-rt
make -C runtime
```

Invoking the CLI through Cargo builds it as part of the command, so the
release CLI cannot be stale for that invocation. If the command exercises
compiled/runtime-linked behavior, refresh both runtime surfaces first --
Cargo does not do that for you.

Use the debug CLI, `cargo run -q -p nova-cli -- ...`, only when you
intentionally want it, for example while debugging Rust-side CLI behavior. It
builds a separate set of large debug artifacts.

## Canonical Full Regression

```bash
scripts/run-nova-full-regression.sh check
scripts/run-nova-full-regression.sh baseline
```

That script already rebuilds:

- the release `nova` CLI
- the release `libnova_async_rt.*`
- `runtime/libnova_runtime.a`

## x86 Container

Canonical container workflow:

```bash
scripts/nova-x86-container.sh build-image
scripts/nova-x86-container.sh start
scripts/nova-x86-container.sh prepare
scripts/nova-x86-container.sh regression check
```

Useful one-offs:

```bash
scripts/nova-x86-container.sh exec 'cargo build -p nova-cli'
scripts/nova-x86-container.sh exec '/tmp/nova-target/release/nova test tests/unit/ --verbose'
scripts/nova-x86-container.sh status
```

Use `prepare` before trusting existing container binaries or runtime artifacts.

## Quick Heuristics

- CLI-only or interpreter-only change: start with `cargo check`.
- Routine Nova package or repo validation: prefer
  `cargo run --release -q -p nova-cli -- ...`, which builds the release CLI as
  part of the command.
- Runtime or compiled-backend change: rebuild release CLI plus runtime
  libraries, or use the canonical full regression script.
- Release-path bug: do not trust debug-only validation.
- Suspicious stale behavior: rerun through the canonical script instead of
  guessing.
