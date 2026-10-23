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
- If you establish that for some multiplicative or additive operator, that one of the operands is actually a constant 0, the operation can naturally be folded to the other operand via the identity rule.
- A pass may find `arith.addi`, identify its operands, determine that the operation is equivalent to `%x`, and replace all uses of the addition's result with `%x` and subsequently erase the redundant operation. 
- Direct transformations provide a great degree of granularity; for example you can instantiate a separate GPU kernel, and rewrite the original's launch site directly.
- However, this degree of granularity means that the author is entirely responsible for managing replacement values, erasure, insertion points, etc. 

##### Pattern rewriting
- MLIR has a dedicated pattern-rewriting framework
- The central idea is to describe a transformation as:
	- *If the IR contains an operation matching this pattern, and the required conditions hold, rewrite it in this particular way*
- A pattern typically derives from `RewritePattern` or one of its more specialised subclasses.
- The reason to use this over direct transformation is that we can use existing infrastructure to make our lives SIGNIFICANTLY EASIER:
	- For example `applyPatternsAndFoldGreedily` is a rewrite *driver* that can be used to apply a set of rewrite patterns to operations until a fixpoint is reached
	- The driver determines operations who match the specified rewrite patterns, and executes them.
	- The infrastructure/drivers will check if ops are safe to delete and will update the worklist correspondingly; no such checks come by default in direct transformations. 
		- Direct transformations, by default e.g., using a C++ iterator, will only visit operations that existed the moment the walk began, meaning that a complex rewrite/pass will miss newly created optimisation opportunities unless a complex queue is manually managed, and this is automated in rewrite drivers.
- In a pass we typically have a pattern container, add our custom rewrite patterns, and then hand them over to a driver (like the one we just mentioned above) to execute over some operations that our pass is permitted to transform. 
- **Canonicalisation** is an important component of this rewriting infrastructure.
- The `canonicalize` pass gathers applicable canoncalisation patterns and applies them to simplify the IR, including removal of redundant operations, simplifying operations, normalising certain operation forms, and constant folding.
	- Merely aims to simplify the current representation. 
### Lowering

### Type conversion


### GPU Compilation full example.
