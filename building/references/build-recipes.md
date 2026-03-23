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

These keep debug Rust artifacts fresh. They do not refresh `target/release/nova`
or the runtime libraries used by compiled/runtime-linked paths.

## Release Surfaces

Release CLI:

```bash
cargo build --release -p nova-cli
```

Runtime artifacts used by compiled/runtime-linked paths:

```bash
cargo build --release -p nova-async-rt
make -C runtime
```

If the next command is `target/release/nova ...`, rebuild the release CLI
first. If the next command exercises compiled/runtime-linked behavior, refresh
both runtime surfaces first.

## Canonical Full Regression

```bash
scripts/full-regression.sh check
scripts/full-regression.sh baseline
```

That script already rebuilds:

- `target/release/nova`
- `target/release/libnova_async_rt.*`
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
- Runtime or compiled-backend change: rebuild release CLI plus runtime
  libraries, or use the canonical full regression script.
- Release-path bug: do not trust debug-only validation.
- Suspicious stale behavior: rerun through the canonical script instead of
  guessing.
