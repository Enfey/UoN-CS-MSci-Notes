### Builder
- `OpBuilder` is the primary C++ object used to create, insert, and manipulate operations, blocks, and regions within an IR.
	- It inherits from `mlir::Builder` which provides helper methods for building context-global objects like types, attributes, and locations, but has no notion of an 'insertion point' so the `OpBuilder` class is used for frontends, which extends with op creation. 
- The **insertion point** is a $(block, position)$ pair, and ops are created immediately before that position.
```
builder.setInsertionPoint(op);            // before op
builder.setInsertionPointAfter(op);
builder.setInsertionPointToStart(block);
builder.setInsertionPointToEnd(block);
builder.setInsertionPointAfterValue(v);   // after v's defining op (or block start if v is a block argument)
```
- Because MLIR nests, the builder constantly dives into regions and corresponding blocks, and visits ops, visits regions we run into an issue
- Say we have a builder constructing operations at block $A$ and temporarily want to build some operations in block $B$.
- Without a **guard** we must manually save and restore the insertion point. 
- For example:
```
// The builder is initially positioned at Point A...

{
	OpBuilder::InsertionGuard guard(builder) 
	
	builder.setInsertionPoint(targetOp); // point B
	builder.create<arith::AddIOp>(loc, lhs, rhs);

} // guard is destroyed, and thus restore point A
```
- This uses C++ RAII pattern: Resource Acquisition is initialisation
	- In this case, the idea is that an objects constructor saves state, and its destructor restores it when the object leaves scope.
	- The destructor for `InsertionGuard` saves the insertion point tuple for the prior insertion, so that it can be restored upon leaving the current block. 

#### Guards in builders in more detail
- We're generating ops using MLIR's `OpBuilder` for some C-like language:
```C
int64_t pick(int64_t n) {
	int64_t r = (n < 2) ? n : n * 2;
	return r + 1;
}
```
- Conceptually, we desire to represent, in order:
	- The function body
	- The ssa value `r` 
	- The arithmetic constant 2
	- The conditional
	- The arithmetic constant `1`
	- The return ssa value
- Now, the addition takes place after, in the enclosing function body block.
- We first build the condition:
```
Value two = builder.create<arith::ConstantIntOp>(loc, 2, 64)

Value cond = builder.create<arith::CmpIOp>(loc, 
	arith::CmpIPredicate::slt, n, two);
```
- Then the corresponding `scf.if` op:
```
auto ifOp = builder.create<scf::IfOp>(loc, TypeRange{i64}
	,cond, /*withElseRegion=*/true);
```
- The result is of type `i64` and naturally denotes `r`, the SSA value we later use in `r+1`
- We fill the separate regions by first constructing a guard to save the builder's current insertion point, which denotes the block and operation:
```
{
	OpBuilder::InsertionGuard guard(builder); // save tuple that is after IfOp
	
	builder.setInsertionPointToStart(ifOp.thenBlock()); // move into the then-block
	
	builder.create<scf::YieldOp>(loc, n);
} // restore the insertion point
```
You then do the same for the else block, so that upon exit, the insertion point is pointing toward, in this case, the terminal instruction in the function body.

If we did not do this, the generated code would be malformed with respect to the source semantics as we never restore our insertion point to the outer enclosing block. Our definition of the arithmetic constant 1 would end up in the else block, along with our return value.

I may revisit this in future, but for now this is generally all I need to be concerned with. 

TableGen will automatically generate the required cpp for a custom-dialect given a valid ODS/.td file, that can then be used with the provided builder to instantiate corresponding MLIR for incoming valid programs.
### Passes/Lowering
- A **pass** is a unit of compiler work applied to an IR operation, commonly a module, function, or another operation.
- Compiler passes may perform optimisations such as local/superlocal optimisations, global inlining, or they may perform certain rewrites, analyse dataflow etc, very general term.
- A pass therefore is a way of organising related compiler transformations and analyses(if they are plural, that is, may not be plural in nature whatsoever).

#### Pass manager
- **MLIR** proves a `PassManager` which runs a configured sequence of passes, handling the orchestration of supplied passes.
	- Naturally, some optimisations/passes depend on certain information being preserved, so it is important to order passes as necessary.
- The `PassManager` orchestrates the top-level pipeline and is associated with an `MLIRContext`, running the pipeline on some root operation, commonly `builtin.module`
	- `OpPassManager` stores an ordered sequence of passes and nested pipelines; the top-level `PassManager` is itself an `OpPassManager`; nested `OpPassManager`'s represent pipelines  to execute on matching nested operations.
		- Typically one creates an `OpPassManager` by explicitly nesting a pipeline within another existing `OpPassManager` via the `nest<OpT>` method, which takes the operation type that the nested pass manager will be constrained to operate on. 
			- This operation type is called a **pass anchor**
			- This is particularly strong, as you can nest pass managers to say, perform optimisations on different functions in parallel.
- `PassManager` automatically parallelises pass execution via multi-threading when passes are independent of one another.
- When MLIR executes a pass, the infrastructure provides a pass instance and a *current operation* relative to the nesting level of the pass manager and the pass anchor.
- An analysis manager is also responsible for performing **analysis**
	- Analysis is related to a pass, but does not transform any code. 
	- Information about control flow, dominance, or other properties useful to transformations or to ensure semantics are preserved, are collected during analyses.
	- The `AnalysisManager` performs analysis lazily and caches analysis results.
	- Analysis for a given operation e.g., `DominanceInfo` may be requested during a transformation, and will thus be computed by the `AnalysisManager`

#### Definition of a pass
- A pass is typically implemented as a C++ class derived from MLIR's pass infrastructure:
```c++
struct MyPass : mlir::PassWrapper<MyPass, mlir::OperationPass<mlir::ModuleOp>>
{
    void runOnOperation() override {
        mlir::ModuleOp module = getOperation();
        // Inspect or transform the module ...
    }
};
```
- Naturally, the pass framework is not restrictive; it does not dictate that every pass must manually walk every operation in its nested context of that anchor.
- A pass may invoke a rewrite driver, defer to the AnalysisManager and subsequent analyses, etc.
- A pass can call `signalPassFailure()` to tell the `PassManager`that it failed.
- MLIR permits developers to 'register' passes so they can be invoked by name through command line tools.

#### Pass factory
- Simply constructs and returns an instance of a pass, typically as `std::unique_ptr<mlir::Pass>`
#### How passes change IR
##### Direct Transformation
- In direct transformation, the pass explicitly finds the operations it wants to change, and performs the necessary modifications itself.
- If you establish that for some multiplicative or additive operator, that one of the operands is actually a constant 1/0, the operation can naturally be folded to the other operand via the identity rule.
- A pass may find `arith.addi`, identify its operands, determine that the operation is equivalent to `%x`, and replace all uses of the addition's result with `%x` and subsequently erase the redundant operation. 
- Direct transformations provide a great degree of granularity; for example you can instantiate a separate GPU kernel, and rewrite the original's launch site directly.
- However, this degree of granularity means that the author is entirely responsible for managing replacement values, erasure, insertion points, etc. 
	- Responsible for ensuring the transformation is semantically valid
	- Ensuring a replacement value has the right type and is available wherever its uses occur
		- And thus, responsible for updating all uses correctly
	- Erasing operations only when safe to do so.
	- The author shoulders a massive amount of responsibility, much of which can be offloaded where suited to *pattern rewriting*

##### Pattern rewriting
- MLIR has a dedicated pattern-rewriting framework
- The central idea is to describe a transformation as:
	- *If the IR contains an operation matching this pattern, and the required conditions hold, rewrite it in this particular way*
- A pattern typically derives from `RewritePattern` or one of its more specialised subclasses.
	- An addition by zero pattern may conceptually look like the following:
		![](Pasted%20image%2020261026204416.png)
- The reason to use this over direct transformation is that we can use existing infrastructure to make our lives SIGNIFICANTLY EASIER:
	- For example `applyPatternsAndFoldGreedily` is a rewrite *driver* that can be used to apply a set of rewrite patterns to operations until a fixpoint is reached
	- The driver determines operations who match the specified rewrite patterns, and executes them.
	- The infrastructure/drivers will check if ops are safe to delete and will update the worklist correspondingly; no such checks come by default in direct transformations. 
		- Direct transformations, by default e.g., using a C++ iterator, will only visit operations that existed the moment the walk began, meaning that a complex rewrite/pass will miss newly created optimisation opportunities unless a complex queue is manually managed, and this is automated in rewrite drivers.
- In a pass we typically have a pattern container, add our custom rewrite patterns, and then hand them over to a driver (like the one we just mentioned above) to execute over some operations that our pass is permitted to transform. 
- It is worth noting that the driver merely maintains some heuristic about how to apply the rewrite patterns, and applies them. 
	- *It does not establish semantic equivalence of the transformed IR.*
- **Canonicalisation** is an important component of this rewriting infrastructure.
- The `canonicalize` pass gathers applicable canonicalisation patterns and applies them to simplify the IR, including removal of redundant operations, simplifying operations, normalising certain operation forms, and constant folding.
	- Merely aims to simplify the current representation. 
	- This `canonicalize` pass, at base, is a general pass that works across multiple dialects.
	- Each operation or dialect can supply its own canonicalisation patterns and folding logic
	- When `canonicalize` runs MLIR gathers the applicable patterns from the loaded dialects and applies them using a greedy rewrite driver.


#### Pass rules
- The restrictions of the pass manager matter; a transformation is not allowed to modify arbitrary IR
- A pass anchored to `@f` is not permitted to rewrite instructions inside `@g`.
- Likewise, a function pass must not erase its parent module or modify the module's body
- These rules are enforced so we maintain a reliable contract that can permit parallel execution and transformation attribution. 
- If a pass really *does* need to modify the enclosing module, then the operation-level design is probably wrong for that task. 
	- For example, a transformation that needs to create a new function at module scope, replace a symbol, and rewrite references across several functions is a natural candidate for a **module-level pass** (OperationPass predicated/anchored on module op)
	- Alternatively, a multi-stage design could separate the work.
		1. Module pass analyses the whole module and identifies what needs changing
		2. A transformation phase rewrites the module and/or individual functions using an appropriate mechanism, ideally with some granularity to achieve parallelism
		3. A later pass validates the result.



### Analysis
- An *analysis* computes information about IR so that a transformation can reuse said information
	- It generally does not transform the IR itself.
- For example: a dominance analysis computes relationships between basic blocks, generally manifesting as analysis of a *control-flow graph*
	- Such information can be needed to establish whether a value is available at a particular use
- Information about control flow, dominance, or other properties useful to transformations or to ensure semantics are preserved, are collected during analyses.
- A custom analysis can be a C++ class constructed for an operation, with a constructor that examines the IR and stores computed results.
- An analysis manager is also responsible for performing **analysis**
	- The `AnalysisManager` performs analysis lazily and caches analysis results.
	- Analysis for a given operation e.g., `DominanceInfo` may be requested during a transformation, and will thus be computed by the analyses governed by `AnalysisManager`
	- This behaves generally similar to `PassManager`; analyses are associated with an operation, analyses can request other analyses creating dependencies
		- A transformative pass can declare that it *preserves* all analyses that have occurred
			`markAllAnalysesPreserved(`
		- Naturally a transformation pass will change the code, and will mean that knowing when an analysis has become invalid is difficult, as we rely on caching.
		- The default assumption is that analyses are invalidated unless a pass explicitly preserves them (or the analysis' own logic determines that its cached info is still valid).
	- The analysis manager manages the lifetime, reuse, caching, and invalidation of reusable information computed by its analyses.
### Verification
- MLIR verification checks whether IR satisfies structural and semantic constraints
- This is restricted merely to the constraints encoded directly in the operation specification in ODS
	- E.g., the types, number of regions, traits, number of terminators with respect to number of blocks, block structure etc.
- If a pass produces an `arith.addi` with incompatible operand types, or constructs an invalid region, verification can detect the problem by comparing against the ODS specification. 
- Verification is not **semantic equivalence checking**
	- A pass could produce perfectly valid MLIR, but the meaning of the lowered/passed variant may be mutated/differ from the original.
- MLIR can be configured to verify IR after each pass, useful for localising passes responsible for introducing invalid IR.
	- This is quite a common methodology.
	- `mlir-opt input.mlir --verify-each`
	- MLIR provides `mlir::verify()` that checks whether the anchor operation and its nested operations satisfy their verification rules.
- You can implement your own verfification
- For properties specific to custom dialects/compilers, one will typically implement custom verification
	- MLIR allows you to implement a `verify()` method for a custom operation
	- For example, suppose we define a custom GPU operation that requires an integer attribute to be positive.
	- You could implement that constraint in the operation's verifier:
```
LogicalResult MyGpuOp::verify() {
  if (getWidth() <= 0)
    return emitOpError("width must be positive");

  return success();
}
```
- Where getWidth() is an accessor either generated from, or implemented for, a custom op definition, retrieving the width attribute belonging to the particular operation instance being verified.
	- This need not be implemented by hand; if one defines ops using ODS, accessors for attributes can be generated automatically.
	- Note that a verifier must also ensure the attribute exists and has the expected type before attempting to use it
- Note that much like other reusable infrastructure with an anchor operation, one can recursively verify ops by specifying an op closer to the top-level of a module(or the module itself.)
	- The custom verifiers for respective operations held by the module will be invoked as part of the top-level invocation. 
#### Limitations of `verify()`
- The main limitation of an ops own `verify()` method is that it is designed merely to check local invariants of that operation, rather than arbitrary properties of the entire compilation. 
- For example, if two operations must agree on some property, e.g., one operation must guarantee it will not mutate `&x` while the other operation holds a reference to `&x`; a custom pass would be more appropriate for this
- For verifying that source operation behaviour is preserved across *lowering* - this is something that requires a dedicated validation component, as this goes far beyond the simple structural checks that `verify()` and `verifyRegions()` and such can perform. 
### Lowering
- **Lowering** is the process of translating a program from a higher-level representation into a lower-level representation, *whilst preserving its intended behaviour*.
- For MLIR, lowering is particularly important because it permits a compiler to represent a program using abstractions that are convenient for optimisation, analysis, and semantics,
- Then one can progressively translate those abstractions into representations that are closer to target hardware.
- For example, at a high-level we may represent a matrix multiplication like so:
```
%result = linalg.matmul ins(%a, %b : ...)
                       outs(%c : ...)
```
- This expresses *what the computation is* without specifying every loop, memory access, affine constraints, machine instructions etc.
- A later lowering may represent the same computation using explicit loops:
```
scf.for %i = %c0 to %M step %c1 {
  scf.for %j = %c0 to %N step %c1 {
    scf.for %k = %c0 to %K step %c1 {
      // Load elements, multiply, accumulate.
    }
  }
}
```
- The exact syntax and sequence will naturally depend on the pipeline. 

#### How do we perform lowering?
- Note that there is no single mandatory lowering path for every MLIR program; a pipeline of passes will be configured appropriate to the input and target.
- A possible CPU-oriented path may see high-level dialects such as `linalg` be lowered to loop and memory representations such as `scf`, `affine`, `memref` etc, with each rewrite being replaced with `LLVM` dialect which can be translated into LLVM IR.
- As we described earlier in the *passes* section, we can define direct or pattern-based rewrites for our dialects, or reuse existing infrastructure to achieve progressive lowering.
### Type conversion
- When an operation is lowered, we must also consider the types of the values it uses and produces.
- An operations representation and its types often need to change together; a high-level type may not have a direct representation in the target dialect.
- MLIR's dialect conversion infrastructure includes a `TypeConverter`, which defines how source types can be represented as target types.
	- It does not, by itself, rewrite every operation that uses the anchored type
	- It merely provides type-conversion rules that other parts of the conversion process use. 

### GPU Compilation example
- A compiler built using MLIR decides which representations to use, which passes to run, and which target backend to invoke.
- We'll follow a GPU kernel that merely adds two arrays element by element:$$C[i] = A[i] + B[i]$$
- We'll be using NVIDIA for this example.
1. We may begin with CUDA C++ Triton, a framework, or some other language.
2. Frontend and initial IR:
	Lex and parse the source, check types (static) and construct an intermediate representation, generally by using the builder pattern
3. High-level MLIR:
	Tensor computations, loops, memory abstractions, `gpu` dialect
4. Lowering and optimisation
	Transform operations and types into representations the target dialect supports
5. Target-specific IR
	`nvvm` in our case
6. GPU code gen
	`nvvm` is a thin-wrapper around NVIDIA's ptx assembly, with some llvm intrinsics; a backend (out of our scope) is responsible for converting `nvvm` to ptx assembly (publicly documented)
	the gpu driver is responsible for converting ptx to SASS (GPU machine code)
7. Execution
- Note the above is a conceptual pipeline, not a universal sequence; some compilers enter MLIR at a higher-level than others, and some use several MLIR dialects, some limit the dialects they use, some skip entire analysis etc, its entirely modular.
#### Source
```
__global__ void add(const float* A, 
				    const float* B, 
				    float *C, 
				    int N) {
	int i = blockIdx.x * blockDim.x + threadIdx.x; // Affine
	if(i < N)
		C[i] = A[i] + B[i];
}
```
- As someone who is generally just familiar with the gpu hardware and has not read much CUDA before, I'll leave this explanation here to future me:
	- We write code in terms of threads and blocks; where a block is a logical abstraction over division into warps, scheduled on a streaming multiprocessor.
	- The number of threads in a block are limited to 1024 on current NVIDIA GPUs - they should be made a multiple of 32 so that every lane is doing some work. 
		- The optimum for a block depends on how many registers and how much shared memory a kernel uses, since those resources limit how many blocks can be scheduled on an SM simultaneously. 
	- Blocks run independently and in no guaranteed order, which is why threads in different blocks cannot synchronise with one another, but threads in the same block/warp can. 
		- Blocks are run on one SM only, and never split, staying there until all threads/warps finish executing. 
		- This is what permits the use of shared (`__shared__`) memory because this is the SM's on chip memory; every block in that thread can thus access it as their warps are scheduled on the same SM; but cannot use `__shared__` between different blocks.
- And so, we take the `blockIdx.x` to determine which block this thread belongs to, we take `blockDim.x` to know how many threads are in each block, and multiply to acquire the total number of threads, up to that point, and then add the thread index to acquire the exact thread executing. 
- BACK TO MLIR!
	- MLIR does not require the frontend to produce a particular dialect
	- A custom compiler may produce a general-purpose representation, a domain specific dialect, etc. 
#### High-level MLIR
- A high-level representation may express the addition in terms of tensors or a structured computation
- At this level, the compiler can reason about the computation as a whole
- It may perform transformations such as loop tiling, fusion etc. 
- This representation exposes useful info e.g., may be able to fuse two tensor operations, avoid writing an intermediate tensor to memory etc.

### Lowering
- Suppose we have the following representation of the kernel:
```
gpu.func @add_kernel(
    %A: memref<?xf32>,
    %B: memref<?xf32>,
    %C: memref<?xf32>,
    %N: index
) kernel {
  %i = gpu.thread_id x
  %a = memref.load %A[%i] : memref<?xf32>
  %b = memref.load %B[%i] : memref<?xf32>
  %sum = arith.addf %a, %b : f32
  memref.store %sum, %C[%i] : memref<?xf32>
  gpu.return
}
```
Okay i cba i get it i'll just read the source.


