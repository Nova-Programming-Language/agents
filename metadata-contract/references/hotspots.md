# Metadata Contract Hotspots

Use this file after classifying whether the bug is in production, install, or
consumption.

## Frontend Contract Production

- `crates/nova-frontend/src/lib.rs`
- module assembly and import compilation paths
- lower-body and validation wiring in frontend lowering code

## Interpreter Contract Consumption

- `crates/nova-interpreter/src/interpreter.rs`
- `set_lowered_module_contract`
- `install_active_module_contract`
- `source_call_target_for_span`
- `call_symbol_with_target`
- `crates/nova-interpreter/src/runtime_symbol_map.rs`
- `crates/nova-interpreter/src/module_loader.rs`

## CLI Paths That Must Install Metadata Before Execution

- `crates/nova-cli/src/commands/run.rs`
- `crates/nova-cli/src/commands/test.rs`
- `crates/nova-cli/src/commands/doc.rs`
- `crates/nova-cli/src/commands/repl.rs`
- `crates/nova-test-runner/src/lib.rs`

## Validation Commands

Use the narrowest reproducer first:

```bash
cargo test -p nova-interpreter lowered_module_contract -- --nocapture
cargo build --release -p nova-cli
cargo run --release -q -p nova-cli -- check path/to/file.nova
cargo run --release -q -p nova-cli -- run path/to/file.nova
```

Cross-command audit after a metadata fix:

- `nova run`
- `nova test`
- `nova doc` example execution
- REPL
- `nova-test-runner`
