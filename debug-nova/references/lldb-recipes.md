# LLDB Recipes For Nova

## Basic Commands

```text
breakpoint set --file crates/nova-interpreter/src/interpreter.rs --line 2371
breakpoint set --name set_lowered_module_contract
run
bt
thread backtrace all
frame variable
frame variable self
expr node_id
next
step
continue
```

## Common Launch Recipes

Interpreter run:

```bash
lldb -- target/debug/nova run path/to/file.nova
```

REPL:

```bash
lldb -- target/debug/nova repl
```

Targeted Nova test:

```bash
lldb -- target/debug/nova test tests/unit/path_or_file.nova --backend interpreter
```

Rust REPL tests:

```bash
lldb -- cargo test -p nova-cli repl_tests -- --nocapture
```

## What To Inspect By Failure Type

Frontend validation failure:

- current module/unit path
- lowered-module validation inputs
- symbol table entries for the failing item
- type catalog and node/call table population around the failing node

Interpreter dispatch failure:

- active module contract
- current span and node ID
- `CallSiteTable` lookup result
- runtime symbol map lookup result
- receiver value and resolved concrete type

REPL state bug:

- hidden wrapper source generated for the current cell
- `compile_and_install` inputs and returned compile result
- interpreter symbol table and installed lowered metadata after the cell

## Practical Rule

If a single breakpoint and `frame variable` can answer the question, do that
before modifying source to add debug prints.
