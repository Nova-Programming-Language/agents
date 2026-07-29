# Memory-Error Detection for Compiled Nova Binaries

Use this reference when a compiled Nova program crashes with SIGBUS, SIGSEGV,
or double-free/UAF symptoms and the cause is not obvious from source. Applies
to binaries produced by `nova build` (C backend or LLVM backend) on macOS.

## When to Reach for libgmalloc

Reach for libgmalloc when:

- a crash reproduces deterministically without the debugger but not under LLDB
  (signal handlers distort timing or intercept faults)
- the stack trace points into ref-count release, free, or atomic decrement
  paths and the faulting address looks like freed memory
- valgrind-style checks are needed on Apple Silicon, where ASAN requires a
  custom toolchain build

libgmalloc places every allocation on its own virtual page and unmaps freed
pages, so use-after-free and buffer-overrun bugs trap immediately at the exact
instruction.

## Basic Invocation

```bash
DYLD_INSERT_LIBRARIES=/usr/lib/libgmalloc.dylib ./your_binary
```

Combine with `MallocStackLogging=1` to record allocation/free backtraces:

```bash
DYLD_INSERT_LIBRARIES=/usr/lib/libgmalloc.dylib MallocStackLogging=1 ./your_binary
```

After a crash, use `malloc_history <pid> <address>` to retrieve the alloc/free
history of the faulting pointer (requires the process to still exist or a
saved log).

Useful extra flags:

- `MALLOC_PROTECT_BEFORE=1` — detect buffer underruns (guard page before)
- `MALLOC_FILL_SPACE=1` — fill with `0x55` to surface uninitialized reads
- `MALLOC_ALLOW_READS=1` — allow over-read of guard page (for diagnostics)

## ARM64 Edge Case: MALLOC_STRICT_SIZE is Incompatible with CoreFoundation

**Do not set `MALLOC_STRICT_SIZE=1` on Apple Silicon.**

`MALLOC_STRICT_SIZE` forces single-byte allocation alignment. On ARM64,
macOS CoreFoundation's `si_item_retain` (invoked transitively from
`__CFInitialize` → `_CFStringGetUserDefaultEncoding` → `getpwuid_r`) uses
atomic `ldadd`/`stadd` instructions that require word-aligned pointers.
Every process crashes during dyld initialization before `main()` runs with:

```
EXC_BAD_ACCESS (SIGBUS) / EXC_ARM_DA_ALIGN at 0x…aff6f
frame 0: si_item_retain
frame 1: si_cache_add_item
frame 3: getpwuid_r
frame 4: _CFStringGetUserDefaultEncoding
frame 5: __CFInitialize
```

The byte-aligned fault address (typically ending in `…f6f`) and the
pre-`main()` backtrace are the signature. This is a documented
libgmalloc/CoreFoundation incompatibility, not a bug in the Nova binary.

Use default vector alignment (16-byte) instead by simply omitting
`MALLOC_STRICT_SIZE`. The libgmalloc banner already warns:

> Applications expecting word-aligned pointers may fail (such as Carbon
> applications). Applications using vector instructions (e.g., SSE) may fail.

## Running libgmalloc Under LLDB

Signal-handler-sensitive crashes often disappear under LLDB. If the crash
only reproduces outside the debugger, reach for libgmalloc before more
elaborate workarounds — the guard-page trap is a hard fault at the exact
instruction, so you do not need to catch a rare signal.

If you still need LLDB with libgmalloc, set env vars inside lldb so they
propagate to the inferior:

```text
env DYLD_INSERT_LIBRARIES=/usr/lib/libgmalloc.dylib
env MallocStackLogging=1
process launch --stop-at-entry
process handle SIGBUS -p true -s true -n true
process handle SIGSEGV -p true -s true -n true
continue
```

`DYLD_INSERT_LIBRARIES` set in the parent shell does not propagate to the
lldb-launched inferior on recent macOS; set it with `env` inside lldb.

## Distinguishing Real Bugs From libgmalloc Artifacts

Before concluding that libgmalloc has surfaced a Nova bug, verify:

1. The crash reproduces on the Nova binary **without** `MALLOC_STRICT_SIZE`.
2. The faulting frame is in Nova runtime or user code, not in
   `libsystem_info.dylib`, `CoreFoundation`, or another system library's
   initializer.
3. The crash is after `main()`, not during dyld init.
4. Symbolize the crash site against the Nova binary's symbols — do not map
   raw offsets against Nova symbols when the instruction pointer lives in
   a system library address range.

If the crash only reproduces with `MALLOC_STRICT_SIZE=1` and the frames are
in system libraries, it is a libgmalloc/CoreFoundation artifact — not a Nova
bug. Re-run with vector alignment to confirm.

## ASAN for Isolated C-Only Components

AddressSanitizer is a better tool than libgmalloc when the bug is confined to
the C runtime (`runtime/src/*.c`) or can be exercised from a standalone C
test binary. ASAN gives allocation/free stack traces and full heap metadata
with lower overhead than libgmalloc, and it does not touch dyld-init
alignment assumptions.

Use ASAN when:

- the bug reproduces from a C-only test harness that links just the C
  runtime sources (no Rust, no compiled Nova output)
- you need alloc/free history for every pointer (libgmalloc requires
  `MallocStackLogging` and is less ergonomic)
- you want leak detection in addition to UAF/overflow

Avoid ASAN when the reproducer exercises:

- the full `nova build` binary (mixes Rust, C runtime, green-thread
  scheduler, and `setjmp`/`longjmp` panic paths — ASAN interceptors
  interact badly with cross-language longjmp and custom stack switches)
- signal handlers installed by the Nova runtime (ASAN installs its own
  SIGSEGV/SIGBUS handler that will mask or conflict with Nova's)

Basic workflow for isolated C components:

```bash
clang -O1 -g -fsanitize=address -fno-omit-frame-pointer \
    -I runtime/include \
    runtime/src/<file>.c test_harness.c -o test_harness
./test_harness   # ASAN errors print with full alloc/free backtraces
```

For leak checking on macOS, ASAN's built-in LSAN is not fully supported on
Darwin — prefer running under Linux (pinned container in `references/
nova-delivery-checks.md`) when leak detection is required.

## Tool Selection Summary

| Situation | Tool |
|-----------|------|
| Bug in full `nova build` binary | libgmalloc |
| Bug in C runtime unit test | ASAN |
| Crash near free/release/atomic decrement | libgmalloc first |
| UAF with alloc-site evidence needed | ASAN if isolatable, else `MallocStackLogging` |
| Leak detection | ASAN in pinned Linux container |
| Signal-sensitive crash that dies outside LLDB | libgmalloc (hard trap at exact instruction) |

## Crash Reports

macOS writes crash reports to:

```
~/Library/Logs/DiagnosticReports/<binary>-<date>.ips
```

The `.ips` file is two concatenated JSON objects (a header line followed by
the body). Parse with:

```python
import json
lines = open(path).read().split('\n', 1)
header = json.loads(lines[0])
body = json.loads(lines[1])
```

Key fields: `exception.signal`, `exception.subtype`, `threads[*].frames`.

Core dumps require root and code-signing entitlements; diagnostic reports
are the practical alternative on stock macOS.
