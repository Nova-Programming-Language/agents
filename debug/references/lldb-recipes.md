# LLDB Recipes

Generic LLDB technique. The project's launch commands (how to start each debug
surface under the debugger, how to build a binary with debug symbols, and the
limits of that debug info) and what to inspect for each failure type are in
the project profile, §Debugging (see `../../references/project-profile.md`).

## Basic Commands

```text
breakpoint set --file path/to/source_file --line 123
breakpoint set --name function_name
run
bt
thread backtrace all
frame variable
frame variable self
expr variable_name
next
step
continue
```

## Launching

Launch any command under LLDB by prefixing it:

```bash
lldb -- <program> <args...>
```

Take `<program> <args...>` from the launch commands in profile §Debugging.

Compiled program with source-line breakpoints: build it with the
debug-symbols command in profile §Debugging, then launch the produced
executable:

```bash
lldb -- /path/to/built-executable
```

Inside LLDB:

```text
breakpoint set --file program_source_file --line 12
run
frame info
frame variable
thread step-over
thread step-in
thread step-out
```

Notes:

- Prefer a build command that leaves a stable executable over a
  build-and-run command, so there is a fixed binary to pass to LLDB.

## What To Inspect By Failure Type

Use the per-failure-type inspection list in profile §Debugging. In general,
inspect the structured data the failing stage consumes (its inputs, the
table or map lookup it performed, and the resolved result) before derived
names or side tables.

## Practical Rule

If a single breakpoint and `frame variable` can answer the question, do that
before modifying source to add debug prints.
