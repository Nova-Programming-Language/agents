# Project Profile

Skills in this directory are project-neutral. Every project-specific command,
path, tool, and convention lives in the project's **profile**: one file in the
project repo that the skills read instead of hardcoding.

## Finding the profile

Use the first that exists, from the repo root:

1. A path named in the repo's `AGENTS.md`, `CLAUDE.md`, or agent rules file
   (look for "project profile").
2. `docs/agent-project.md`

If there is none, ask the user for the commands a step needs. Never guess a
build, test, or regression command.

## Sections

A skill that needs a project fact names the profile section that holds it, as
"profile §Build". Profiles use these `##` headings, and leave out sections
that don't apply. A skill that finds a needed section missing asks the user.

| Section | Holds |
|---|---|
| Paths | Where specs, architecture docs, the architecture map (which also names the components), the semantic-families doc, an exemplar architecture doc, project notes, reports, status files, and logs live |
| Rules | The project's rules file, authority order (which document wins on conflict), and language-semantics notes agents must keep in mind |
| Build | Build commands for each artifact, rebuild/clean steps, artifact locations, how to tell an artifact is stale |
| Environments | Pinned containers or remote hosts, and how to run builds and tests in them |
| Tests | Test layout and kinds, how to run one test, a file, a component suite; test file conventions and annotations |
| Regression | Full-regression and repo-verification commands, baselines, where results and failure lists are written, gates |
| Command surfaces | User-facing commands a behavior change must stay consistent across |
| Packages | Package manager commands and the package onboarding checklist |
| Debugging | Debugger entrypoints, symbol/breakpoint maps, memory-error tools, runtime evidence to gather |
| Formatting | Formatter commands and what scope they may touch |
| Docs | Doc-comment conventions and doc generation/validation commands |
| Delivery checks | Project-specific questions to answer before calling work done |
| Repository | Remote, default branch, issue tracker and labels, PR conventions |

## Writing a profile

State commands exactly, as they are run from the repo root. Prefer pointing to
a script over repeating its flags. Keep rationale in the project's own docs;
the profile is a reference, not a tutorial.
