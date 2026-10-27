# Overview
- The `gpu` dialect organises gpu related mid-level abstracted operations, types, attributes, dataflow analyses, and transformation rules.
- Naturally, it defines its own domain rules but leverages MLIR's core framework features e.g., debug info tracking discussed in the [MLIR Overview](MLIR%20Overview.md).
- To begin with, we'll focus only on the collective operations. As Eidolon potentially expands, we may extend to matrix ops, execution-model queries and other gpu capabilities.

## `gpu` dialect
### `gpu.shuffle`
- **Description**
#### TableGen
#### Lowering
##### `nvvm`
##### `rocdl`
##### `SPIR-V`
### `gpu.subgroup_reduce`
- **Description**
#### TableGen
#### Lowering
##### `nvvm`
##### `rocdl`
##### `SPIR-V`
### `gpu.subgroup_broadcast`
- **Description**
#### TableGen
#### Lowering
##### `nvvm`
##### `rocdl`
##### `SPIR-V`
### `gpu.ballot`
- **Description**
#### TableGen
#### Lowering
##### `nvvm`
##### `rocdl`
##### `SPIR-V`