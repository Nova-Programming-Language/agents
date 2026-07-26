# Package Repository Onboarding

Apply this checklist before preparing, building, testing, modifying, or
delegating work for a package repository.

Read, in order:

1. Every applicable `AGENTS.md`, from repository root to the target package.
2. The repository's agent-rules document referenced by `AGENTS.md`.
3. The repository-root `README.md`.
4. The target package's `README.md`, when present.
5. The target package's `docs/README.md`, when present.
6. Any README belonging to a nested example or test target in scope.

Package READMEs commonly define live-service prerequisites, native library
versions, environment variables, preparation commands, and which tests are
expected to run. Treat those instructions as required workflow context unless
they conflict with a higher-authority specification or agent rule.

If an expected README is absent, state that explicitly and continue with the
remaining sources. Do not claim README compliance unless the applicable files
were actually read.

For delegated work, put the exact onboarding files in the worker brief. Before
accepting a worker result, require it to report which README files it read and
whether its commands matched their instructions. If work began without this
onboarding, audit the completed commands against the READMEs before proceeding.
