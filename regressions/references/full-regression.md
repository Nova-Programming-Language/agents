# Regression Workflows

Use this file after deciding whether you need targeted validation or the
canonical full-suite tracker. The concrete commands, flags, paths, and
artifact names are in the project profile
(`../../references/project-profile.md`): test commands in §Tests, full
regression in §Regression, containers in §Environments.

## Targeted Validation

Use one of three command families: targeted file/filter runs, directory/subtree
runs, or canonical full regression through the full-regression script
(profile §Regression).

For the first two, check profile §Tests for whether the test runner records
scoped results by default; use its explicit export or artifact-directory
options only for exports or overrides.

Kinds of targeted validation, each with commands in profile §Tests:

- **Component/unit validation**: compile check plus filtered unit tests for
  the changed component or module.
- **Targeted test-runner validation**: one directory, one file, or one
  name filter, optionally pinned to a single engine or backend.
- **Directory/subtree validation**: a subtree of the test tree.
- **Compiled differential validation**: refresh the runtime libraries first
  (`../../references/artifact-freshness.md`), then run the tests in the mode
  that exercises every engine. Read per-engine outcomes and divergence from
  the structured report fields profile §Tests names, never from the
  human-readable message.
- **Failure investigation**: rerun the last failures, or replay them from the
  last recorded snapshot without executing.

Scoped run notes:

- use what the runner prints to confirm exact scope and stored paths
- replaying the last failures reads the current scope's snapshot only
- a scoped run over the whole test or examples tree is still a scoped run,
  not canonical full regression

## Canonical Full Regression

Primary commands (profile §Regression): run-and-compare (optionally marked as
a candidate with a note), the failure view, the run list, run detail,
run-and-promote baseline, and promote-an-existing-run as baseline.

Core semantics to confirm in profile §Regression:

- a check run records an immutable full run and compares it against the
  baseline and other reference runs
- the failure view is the primary machine-readable failure output
- the run list and run detail recover cross-session run context
- promoting a recorded run to baseline does not rerun it
- per-case failure artifacts are stored alongside the suite reports; prefer
  those over the full run log when drilling into one failing case

Run references (named runs such as the last run, the previous run, the last
clean run, the baseline, the best recorded run) are listed in profile
§Regression.

Default agent loop:

1. run the full-regression check
2. read the failure view
3. pivot to targeted reruns
4. promote a baseline only deliberately, from a named recorded run

## Starting a Long Run Detached (survives task kills)

A full run can take tens of minutes or more (profile §Regression gives the
expected duration). Two failure modes lose the run if it is launched
carelessly, both observed 2026-08-29:

1. Background Bash calls are killed at the ~10-minute cap **by process
   group**, so a `nohup`'d job launched inside the same
   `run_in_background` call as its watcher dies with the watcher (nohup
   only shields SIGHUP). Never bundle the job and its watcher in one call.
2. Silent background sleep-loop watchers get reaped as idle even when
   bounded below the cap.

The pattern that survives:

```bash
# 1. Launch in a QUICK FOREGROUND Bash call (returns immediately; the
#    child survives across tool calls on macOS — setsid does not exist there).
#    <gate command> is the full-regression gate from profile §Regression.
nohup bash -c '<gate command>; \
  echo "GATE_EXIT_CODE=$?"' > "$SCRATCHPAD/full-gate.log" 2>&1 < /dev/null &
disown $!
echo $! > "$SCRATCHPAD/full-gate-pid"
```

2. Watch with the **Monitor tool**, not a background sleep loop: a poll
   loop (sleep 30) that emits progress (the progress signal profile
   §Regression names, such as completed shards against the shard total),
   fires immediately on failure signatures (the failure strings profile
   §Regression lists), and exits with a terminal event when the
   pid dies, printing the log's last `GATE_EXIT_CODE` line and verdict
   line. Set the Monitor timeout to the full expected duration (up to
   60 minutes); re-arm on expiry.

3. The `echo "GATE_EXIT_CODE=$?"` trailer gives the watcher an
   unambiguous terminal signature; cover failure states in the filter,
   not just success.

If the session loses the log anyway, the run itself is durable when the
regression tool records runs: the recorded run directory (profile
§Regression) plus the run list and run detail views recover everything, and
the progress signal tells you how far it got.

## Comparison Verdict Versus Absolute Result

These answer different questions, and only one of them is what a reader assumes
"the regression passed" means:

- **Comparison**: new, fixed, and unchanged failure counts in the
  baseline-comparison artifact, and the verdict (regression, clean,
  stable-failing, no-baseline). All are relative to the recorded baseline.
- **Absolute**: the total failure count, the per-suite counts in the run
  manifest, and — when it follows the total — the process exit status.

Profile §Regression names the artifacts and fields for each, and says which
one the exit status follows.

Two consequences worth holding onto:

- A run whose long-standing failures are already recorded in the baseline
  reports zero new failures and a non-regression verdict. That is a correct
  comparison result; it is not a clean suite.
- A missing baseline is a bookkeeping state, not a failure. Promote a baseline
  when you want comparisons, not to make a run pass.

State the absolute failure count whenever you report a regression result, and
identify the known-failing cases by name so they stay visible. When the
question is whether anything got worse, read the new-failure count from the
comparison artifact rather than the exit status.

## Full Regression Artifacts

Profile §Regression lists the stable artifact locations (reference-run map,
last-run manifest, last-run failures, comparisons against baseline, previous,
and best-recorded runs, and per-run directories), the structured failure
record format, the meaning of each field, and how to rerun one record.

Structured failure entries carry enough context to classify, triage, and fix
failures without parsing raw compiler output. Expect fields covering
classification relative to the comparison target, canonical case identity,
status, message, likely owning subsystem, failing phase, per-case artifact
paths. How to rerun a single failure from its record is in profile
§Regression.

## Reading Failure Records

Keep the project's test-system model in mind (profile §Regression): what
identifies a test case canonically, which statuses count as failing, and
whether one test file may legitimately produce several cases (for example
across engines or forms).

- Each failure record carries the fields profile §Regression lists, typically
  a classification, case identity, status, message, likely owner, failing
  phase, and per-case artifacts.
- Classifications are relative to the chosen comparison target, typically the
  baseline.
- The run log is archival; per-case artifacts are the place to drill into one
  failing case.

## Pinned Container Path

Container build, start, prepare, and regression commands are in profile
§Environments.

Use this when reproducibility on the pinned platform matters.
