---
name: metadata-contract
description: Inspect or debug Nova lowered metadata contracts; use for LoweredModule, TypeCatalog, NodeTypeTable, CallSiteTable, symbol-map, and contract-validation issues.
---

# Metadata Contract

## When to Use

Use this skill when the issue is about lowered metadata production or
consumption rather than general language semantics.

Typical triggers:

- `LoweredModule::validate()` failures
- missing or wrong `NodeTypeTable` or `CallSiteTable` entries
- symbol resolution disagreements between frontend and interpreter
- interpreter behavior that still looks like fallback inference
- reviews of metadata-migration work against the contract docs

## Source of Truth

Read these in order:

1. `docs/architecture/backend-metadata-contract.md`
2. `docs/architecture/interp-backend-architecture.md`
3. `docs/architecture/interp-backend-evolution-plan.md`
4. `docs/architecture/frontend-evolution-plan.md`
5. `../references/source-of-truth.md`
6. `references/checkpoints.md`
7. `references/hotspots.md`

The contract document is normative for schema and invariants. The interpreter
architecture defines backend conformance. The evolution plans explain which
phase a gap belongs to.

## Non-Negotiable Rules

- `LoweredModule` is the authoritative frontend/backend boundary.
- The interpreter must execute metadata from the lowered module, not re-infer
  types, calls, or symbols from surface AST.
- Missing contract data is a bug, not a signal to add a fallback.
- `ConcreteTypeId` lookups must resolve through `TypeCatalog`.
- Call-like operations must route through `CallSiteTable` plus the runtime
  symbol map.

The architecture explicitly requires no fallback inference and hard failure on
contract violations.

## Forbidden Fallback Patterns

- If `CallSiteTable` is required, do not fall back to callee-name matching,
  AST inspection, or legacy call-target lookup.
- If `NodeTypeTable` is required, do not infer from operators, literals,
  registries, or surface syntax.
- If runtime symbols should come from lowered metadata, do not synthesize them
  from strings or side tables.
- If `LoweredModule::validate()` should reject the state, do not patch the
  interpreter to accept it.
- If contract data should exist but does not, classify it as a producer or
  install bug, not a consumer invitation to guess.

## Triage Workflow

1. Classify the failure:
   - frontend production bug
   - lowered-module validation bug
   - interpreter install bug
   - runtime dispatch or type-driven decision bug
2. Confirm which phase the work claims to implement.
3. State the exact answer the runtime is trying to compute and which contract
   table owns that answer.
4. Inspect the metadata object before modifying runtime behavior.
5. Use `references/checkpoints.md` to verify the relevant contract tables and
   install path.
6. Only patch the interpreter if the frontend already supplies the needed data
   and the interpreter is consuming it incorrectly.
7. If the data is absent and should exist, stop and fix or report the producer
   or installer path instead of adding fallback logic.

When fixing a metadata issue, add or update tests that prove missing required
contract data fails loudly rather than silently falling back to non-contract
inference.

## Open These References As Needed

- `../references/source-of-truth.md`
- `references/checkpoints.md`
- `references/hotspots.md`
