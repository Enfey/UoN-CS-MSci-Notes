> An intermediate representation (IR) is a machine-agnostic representation placed between front-end(parsing+semantics) and back-end(codegen).

A well formed IR permits one to:
- Apply optimisations cleanly (e.g., constant folding, DCE)
- Separate language concerns from machine concerns, 
- Make retargeting easier; many backends can consume the same IR. 