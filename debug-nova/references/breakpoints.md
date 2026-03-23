# Breakpoint Map

Use this file after choosing the debug surface.

## Frontend Compile And Validation

- `crates/nova-frontend/src/lib.rs:353` `compile`
- `crates/nova-frontend/src/lib.rs:2240` `compile_single_module`
- `crates/nova-frontend/src/lib.rs:2361` `compile_module`
- `crates/nova-frontend/src/lib.rs:2746` `resolve_imported_module_impl_source`

## Interpreter Metadata Install And Dispatch

- `crates/nova-interpreter/src/interpreter.rs:2371`
  `set_lowered_module_contract`
- `crates/nova-interpreter/src/interpreter.rs:2454`
  `install_active_module_contract`
- `crates/nova-interpreter/src/interpreter.rs:3043`
  `source_call_target_for_span`
- `crates/nova-interpreter/src/interpreter.rs:3109`
  `call_symbol_with_target`
- `crates/nova-interpreter/src/interpreter.rs:3279` `load_prelude`

## Import And REPL

- `crates/nova-interpreter/src/module_loader.rs:122`
  `compile_loaded_source`
- `crates/nova-cli/src/commands/repl.rs:88` `compile_and_install`
- `crates/nova-cli/tests/cli_tests.rs:2983` `repl_tests`
