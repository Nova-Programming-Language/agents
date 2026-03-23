# Regression Workflows

Use this file after deciding whether you need targeted validation or the
canonical full-suite tracker.

## Targeted Validation

Rust crate validation:

```bash
cargo check -p nova-cli
cargo test -p nova-cli <filter>
```

Targeted Nova interpreter/test-runner validation:

```bash
cargo run -- test tests/unit/ --skip-compiled
cargo run -- test tests/compile_errors/
cargo run -- test tests/unit/math_tests.nova
```

Compiled differential validation:

```bash
cargo build --release -p nova-async-rt
make -C runtime
cargo run -- test tests/unit/ --compiled
```

CLI/test-runner failure investigation:

```bash
cargo run -- test --failed
cargo run -- test --failed --from-last
```

## Canonical Full Regression

Primary commands:

```bash
scripts/full-regression.sh check
scripts/full-regression.sh check --candidate --note "ready for review"
scripts/full-regression.sh failed
scripts/full-regression.sh list
scripts/full-regression.sh show
scripts/full-regression.sh baseline
scripts/full-regression.sh baseline set last-run
```

Core semantics:

- `check` or `run` records an immutable full run and compares it against
  `baseline`, `previous-run`, and `best-recorded`
- `failed` is the primary machine-readable failure view
- `list` and `show` recover cross-session run context
- `baseline set <run-ref>` promotes a recorded run without rerunning

Run references:

- `last-run`
- `previous-run`
- `last-clean`
- `baseline`
- `best-recorded`

Default agent loop:

1. `scripts/full-regression.sh check`
2. `scripts/full-regression.sh failed`
3. pivot to targeted reruns
4. promote only with `baseline set <run-id>` when intended

## Full Regression Artifacts

Stable aliases under `tests/.nova/full-regression/`:

- `refs.json`
- `last-run-manifest.json`
- `last-run-failures.json`
- `last-run-vs-baseline.json`
- `last-run-vs-previous.json`
- `last-run-vs-best-recorded.json`
- `runs/<run-id>/`

## x86 Container Path

```bash
scripts/nova-x86-container.sh build-image
scripts/nova-x86-container.sh start
scripts/nova-x86-container.sh prepare
scripts/nova-x86-container.sh regression check
```

Use this when Linux/x86_64 reproducibility matters.
