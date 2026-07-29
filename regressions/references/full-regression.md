# Regression Workflows

Use this file after deciding whether you need targeted validation or the
canonical full-suite tracker.

## Targeted Validation

Use one of three command families: targeted file/filter runs, directory/subtree
runs, or canonical full regression through `scripts/run-nova-full-regression.sh`.

For the first two, `nova test` already records exact-scope local state by
default. Use `--report-json` or `--artifacts-dir` only for explicit exports or
overrides.

Rust crate validation:

```bash
cargo check -p nova-cli
cargo test -p nova-cli <filter>
```

Targeted Nova interpreter/test-runner validation:

```bash
cargo run -- test tests/unit/ --backend interpreter
cargo run -- test tests/compile_errors/
cargo run -- test tests/unit/math_tests.nova
cargo run -- test tests/unit/ --filter select
```

Directory/subtree validation:

```bash
cargo run -- test tests/unit/concurrency/ --backend interpreter
cargo run -- test tests/build/
```

Compiled differential validation:

```bash
cargo build --release -p nova-async-rt
make -C runtime
cargo run -- test tests/unit/ --backend c
```

CLI/test-runner failure investigation:

```bash
cargo run -- test --failed
cargo run -- test --failed --from-last
```

Scoped `nova test` notes:

- use the printed `Scope` and `Recorded` blocks to confirm exact scope and
  stored paths
- `--from-last` reads the current exact-scope snapshot only
- `nova test tests/` and `nova test examples/` are still scoped `nova test`
  runs, not canonical full regression

## Canonical Full Regression

Primary commands:

```bash
scripts/run-nova-full-regression.sh check
scripts/run-nova-full-regression.sh check --candidate --note "ready for review"
scripts/run-nova-full-regression.sh failed
scripts/run-nova-full-regression.sh list
scripts/run-nova-full-regression.sh show
scripts/run-nova-full-regression.sh baseline
scripts/run-nova-full-regression.sh baseline set last-run
```

Core semantics:

- `check` or `run` records an immutable full run and compares it against
  `baseline`, `previous-run`, and `best-recorded`
- `failed` is the primary machine-readable failure view
- `list` and `show` recover cross-session run context
- `baseline set <run-ref>` promotes a recorded run without rerunning
- full regression stores per-case failure artifacts alongside the suite reports;
  prefer those over `full-run.log` when drilling into one failing case

Run references:

- `last-run`
- `previous-run`
- `last-clean`
- `baseline`
- `best-recorded`

Default agent loop:

1. `scripts/run-nova-full-regression.sh check`
2. `scripts/run-nova-full-regression.sh failed`
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

Structured failure entries carry enough context to classify, triage, and fix
failures without parsing raw compiler output. Example record from
`failed --json`:

```json
{
  "classification": "new_in_last_run",
  "case_id": "interpreter::unit::standard::tests/unit/math_tests.nova::integer_overflow",
  "engine": "interpreter",
  "form": "unit",
  "tier": "standard",
  "file": "tests/unit/math_tests.nova",
  "case_name": "integer_overflow",
  "source_case_key": "math_tests::integer_overflow",
  "status": "fail",
  "message": "expected 0, got 4294967295",
  "owner": "frontend",
  "phase": "execution",
  "artifacts": [
    "runs/20260329-143012/cases/interpreter-unit-standard-math_tests-integer_overflow/stderr.txt",
    "runs/20260329-143012/cases/interpreter-unit-standard-math_tests-integer_overflow/diff.txt"
  ],
  "rerun_hint": "cargo run -- test tests/unit/math_tests.nova --filter integer_overflow"
}
```

Field reference:

| Field | Meaning |
|---|---|
| `classification` | Relationship to comparison target — `new_in_last_run`, `persistent`, `fixed_in_last_run`, `improvement` |
| `case_id` | Canonical identity: `engine::form::tier::file::case_name` |
| `engine` | `interpreter` or `compiled` |
| `form` | `unit`, `program`, or `diagnostic` |
| `tier` | `standard` or `e2e` |
| `file` | Test source file path |
| `case_name` | Individual test case within the file |
| `source_case_key` | Source-level key (module::case) for cross-engine matching |
| `status` | `fail`, `not_compilable`, `discovery_error`, or `infra_error` |
| `message` | Human-readable failure summary |
| `owner` | Subsystem that likely owns the defect: `frontend`, `interpreter`, `c-backend`, `runtime`, `tests`, `cli` |
| `phase` | Where execution failed: `compilation`, `execution`, `discovery`, `infra` |
| `artifacts` | Per-case failure files (stderr, diff, IR dumps) under the run directory |
| `rerun_hint` | Exact command to reproduce this single failure |

The output of `failed --json` is a JSON array of these records.

## x86 Container Path

```bash
scripts/nova-x86-container.sh build-image
scripts/nova-x86-container.sh start
scripts/nova-x86-container.sh prepare
scripts/nova-x86-container.sh regression check
```

Use this when Linux/x86_64 reproducibility matters.
