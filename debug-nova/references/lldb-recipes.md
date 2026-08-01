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

Compiled C-backend program with Nova source-line breakpoints:

```bash
cargo run -p nova-cli -- build path/to/program.nova --backend c --debug-symbols -o /tmp/nova-debug-probe -v
lldb -- /tmp/nova-debug-probe
```

Inside LLDB:

```text
breakpoint set --file program.nova --line 12
run
frame info
frame variable
thread step-over
thread step-in
thread step-out
```

Notes:

- Prefer `nova build` over `nova run` for debugger sessions because `build`
  leaves a stable executable to pass to LLDB.
- `--debug-symbols` is C-backend-only. It emits debuggable C object flags and
  source `#line` directives; on macOS it also emits a `.dSYM` bundle.
- Treat this as line-number/source-location debugger support, not full Nova
  variable debug info. Source-backed params and locals may be visible as C
  locals, while backend temporaries and module globals use generated C symbols.
- `thread step-in` can stop once in a generated prologue pseudo-file before
  reaching the first Nova statement; one `thread step-over` usually reaches the
  Nova source line. Source breakpoints and function step-over should stay on
  Nova lines.

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
