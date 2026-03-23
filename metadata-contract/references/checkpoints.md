# Metadata Contract Checkpoints

## Frontend Production Checkpoints

- Does the produced `LoweredModule` validate successfully?
- Are all imported and entry units present in `LoweredModule.units`?
- Does each referenced `ConcreteTypeId` exist in `TypeCatalog`?
- Are binding and expression node IDs populated in `NodeTypeTable`?
- Does every call-like node have a `CallSiteTable` entry?
- Are symbols present in `SymbolTable` with the expected unit ownership?

## Interpreter Consumption Checkpoints

- Is `set_lowered_module_contract` called before execution?
- Is the active unit contract installed for the current source path or REPL
  cell?
- Does the runtime symbol map contain the symbol referenced by the call site?
- Is dispatch using call metadata rather than legacy name/type inference?
- Are type-driven decisions reading `TypeCatalog` rather than string matches?

## Review Questions

- Is this change actually phase-appropriate, or is it solving a later phase?
- Does the implementation add a fallback around missing metadata?
- Would the same bug show up in `run`, `test`, `doc`, `repl`, or the test
  runner because one of them still installs metadata differently?
- Is the failure really interpreter-side, or is the frontend producing invalid
  or incomplete metadata?
