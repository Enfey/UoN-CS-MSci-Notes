### Top-level
- MLIR is **reusable compiler infrastructure** which permits someone to implement their own semantics on top.
- If you wanted to construct a cutting-edge compiler for a new language, it would be laborious to write your own parser infrastructure, expressive intermediate representations, graph manipulation utilities, rewrite engine, dataflow analysis, code-generation and optimisation, etc
- MLIR provides many of the above for free, only requiring the ability to target the dialect you define. 

### Compiler IR
> An IR is a machine-agnostic representation placed between front-end and back-end.
- Naturally these are denoted at different levels of abstraction to permit certain optimisations, analyse/annotate the program, and progress toward code-generation.
- The structural organisation of an IR has generally has a strong impact on how the compiler considers analysis, optimisation, and code generation. 
	- E.g., treelike/graphlike IRs will naturally lead to passes structured as some form of treewalk. Similarly linear IRs (representing machine-agnostic pseudocode e.g., SSA naturally lead to passes/partial passes that iterate over operations in-order)
- Common intermediate representations include:
	- **Directed Acyclic Graph**
	- **SSA**
	- **Control-Flow Graph**
	- More on this stuff [Lecture 6+7 - Intermediate Representationms](Lecture%206+7%20-%20Intermediate%20Representationms.md)
### Multi-Level Intermediate Representation
- MLIR's intermediate representation is what we'll call composable.
- That is, one can define their own elements that correspond to MLIR's general form.
#### Operations
- **Operations** are the central unit of computation and structure in MLIR. 
- Everything from simple multiplication to a complete source module or a function declaration is represented as an operation
- An operation is fully open and extensible, and consists of the following components:
		- **Opaque name** - prefixed by the dialect identifier
		- **Operands** - some inputs consumed by an operation
		- **Results** - SSA value produced by the operation
		- **Attributes** - metadata attached to an operation or another IR object, this can be literally anything and is quite powerful for representing semantics.
		- **Regions** - see brlow
		- **Source location**
- An example set of operations is depicted below:
```
module
func.func @calculate(%x: i32, %y: i32) -> i32 {
		%sum = arith.addi %x, %y : i32
		return %sum : i32
	}
}
```
- `builtin.module` represents the module we are concerned with.
- `func.func` represents the function
- `arith.addi` represents integer addition
- `func.return` represents returning from the function
- Such operations have different roles but will naturally reuse the existing IR infrastructure.
##### Operation names and dialects
- An operation's name generally has the form: `dialect. operation`
- This name naturally corresponds to a definition that constrains its usage and generation (e.g., when lowering).

##### Operands and results
- Operands are inputs consumed by an operation. This may be another operation, or something else entirely.
- Results are SSA values produced by an operation
	- See [Lecture 6+7 - Intermediate Representationms](Lecture%206+7%20-%20Intermediate%20Representationms.md)

##### Attributes
- Attributes hold information associated with operations or other IR objects.
- Common examples include integer or string values, symbolc references, and target configuration for lowering. 
- For example:
```MLIR
%result = arith.cmpi slt %x, %y: i32
```
- The `slt` comparison predicate is used to check if the first signed integer operand is strictly smaller than the second signed integer operand.
- Some operations require specific attributes to dictate their behaviour/semantics. 

###### Examples of default attributes in MLIR that can be used in custom operation definition(s)
- **Scalar/Basic**
	- `IntegerAttr` holds a compile-time, signless, signed, or unsigned integer value:
		- E.g., `%cMAX = arith.constant 2147483647: i32`
		- Where `arith.constant` takes a `value`  as its parameter. This value can be any typed attribute e.g., `FloatAttr, DenseElementsAttr` and so on.
	- `FloatAttr`
		- Represented in a similar manner to the above.
		- A custom operation could use floating point attribute to specify a fixed-scale or coefficient `mydialect.scale %input {factor = 0.5 : f32}`
	- `BoolAttr`
	- `StringAttr`
	- `UnitAttr`
- **Collection & Structural**
	- `ArrayAttr`
	- `DictionaryAttr`
	- `DenseArrayAttr`
	- `AffineMapAttr`
	- `IntegerSetAttr`
- **Aggregate Data Attributes**
	- `DenseElementsAttr`
	- `SparseElementsAttr`
	- `DenseResourceElementsAttr`
- **Type and Symbol references**
	- `TypeAttr`
	- `SymbolRefAttr/FlatSymbolRefAttr`
- Notably, Attributes are described here in terms of their actual attribute classes that exist in the MLIR specification. 
- In ODS(further below), we refer to more specific attribute specifiers that place additional constraints on the generic ones referred to above e.g., `F32Attr` which gives you an additional type check.
	- But you can define your own attributes, using these basic ones.
	- `def F32Attr : FloatAttrBase<F32, "32-bit float attribute">;` for example.
Will just deal with these as we come across them. One important attribute is the `OptionalAttr<Attr>` which can be left completely blank. 


##### Regions
- A ***region*** is a list of ***blocks*** owned by an op
###### Blocks
- A **block** an ordered list of operations.
	- The last operation in a block must be a **terminator** (an operation that handles control flow, such as branch, conditional branch, or return) (unless opt-out?)
	- MLIR uses **block arguments** to pass values into a block from a predecessor.
		- This is opposed to the Phi functions present in SSA form; instead of the destination block choosing a value based on where it came from, MLIR predecessors explicitly push the required arguments forward during a branch operation. 
- Every block belongs to a parent `Region`, which in turn is nested inside a parent `Operation`.
- An MLIR block itself is not an SSA value and does not have a type.
	- There are operations/methods one can perform over a block: TBC
- A block is prefixed with `^`, followed by a name.
- A symbol is denoted via `@` - symbols are defined inside a region with a `SymbolTable`, used for global variables, recursion, linking etc.
	- Standard SSA values require a direct parent-child or dominator-tree dataflow path. Symbols resolve issues like global vars for linking etc.
- A dialect defined type is denoted via `!`

###### Regions
- A region is an **ordered list of blocks**, with the first block being the entry block, where execution of the region starts.
- The upward chain of op->block->region->op->block->region continues until one reaches the top-level `module`, which has no parent. 
- This makes enough sense, so here we'll talk about the notion that an operation can own a region
- MLIR permits one to keep an operation, say a loop, as a single operation that owns its own body, in the form of a region.
	- The loops bounds, body, and its results are all attached to one op.
```
func.func @sum(%buf: memref<?xf32>, %n: index) -> f32 {
  %c0 = arith.constant 0 : index
  %c1 = arith.constant 1 : index
  %zero = arith.constant 0.0 : f32
  %sum = scf.for %i = %c0 to %n step %c1 iter_args(%acc = %zero) -> (f32) {
    %v = memref.load %buf[%i] : memref<?xf32>
    %next = arith.addf %acc, %v : f32
    scf.yield %next : f32
  }
  func.return %sum : f32
}
```
- Define a named function `@sum`
- This function owns one region: the function body
- That region has one block, the entry block. 
- The terminator returns the ssa value to the caller
- `scf.for` is itself an operator in the block that maintains its own region, containing one block holding 3 operations. 
- We can reveal the implicit block by observing the generic form like so:
```
%sum = "scf.for"(%c0, %n, %c1, %zero) ({
^bb0(%i: index, %acc: f32):
  %v = "memref.load"(%buf, %i) : (memref<?xf32>, index) -> f32
  %next = "arith.addf"(%acc, %v) <{fastmath = #arith.fastmath<none>}> : (f32, f32) -> f32
  "scf.yield"(%next) : (f32) -> ()
}) : (index, index, index, f32) -> f32
```
-` ({})` Denotes the start of a region owned by `scf.for`

- Regions can be defined concretely in ODS, next to `arguments` and `results` specfiying how many regions the op can ownand how many blocks the region(s) contain.
- For example:
	- `let regions = (region SizedRegion<1>:$body)
		- Op owns one region called `body` which must contain one block
		- Other options include `AnyRegion` specifying that a region can hold any number of blocks, and `VariadicRegion`, which permits a variable number of regions. 
		- You can use traits when specifying the op to denote to give all regions implicit terminators, denote op's side effects in relation to its regions etc.
##### Source locations
- When outputting to `.mlir` each op will carry its source file, line, and column. 
- Locations are hidden when IR is printed normally to reduce clutter, but they can be enabled with `--mlir-print-debuginfo`
- The compiler frontend while generating IR will turn AST node(generally speaking)positions into a location and pass it when instantiating the IR.

```
Location loc = FileLineColLoc::get(builder.getStringAttr(file), line, col);
builder.create<TransposeOp>(loc, input);
```
- This is used in errors, and is used in transformations e.g., rewrite pattern replaces an op, it gives the new ops the old ops location. 
- Merges combine locations; if a pass fuses two ops into one, it can give the result a `FusedLoc` containing both source file and line origins, so neither is lost.
- Inlining wraps the original location, specifying the inline location, and what was inlined specifically by denoting the OG location. 
- Source locations are mandatory in MLIR.

#### Operation formats, and defining custom operations and attributes
- Naturally, both operation formats are equivalent.
##### Generic Operation Format
```
%result = "arith.addi"(%a, %b) : (i32, i32) -> i32
```
- This is the hyper-strict variant of operations that one will encounter only if you force the compiler driver to print this variant, or if you write this to a `.mlir` file yourself with your compiler frontend.
- The operation name is specified in quotes, with operands then specified in brackets. 
- A functional type signature explicitly maps the inputs to outputs
- We would specify attributes in curly braces as a dictionary prior to denoting the functional relation. 
##### Custom Operation Format
```
%result = arith.addi %a, %b : i32
```
- Much simpler and intuitively readable, as not everything is made explicit.
- Attributes may look different here. 
	- The attribute value is often inlined directly, with the braces omitted.
```
%0 = llvm.mlir.constant(420 : i64) : i64
```
##### Defining Operations
- Defining Operations is done using **ODS** - the **Operation Definition Specification**
- This is powered by LLVM's **TableGen** utility (`.td` files)
- TableGen permits one to define an operation declaratively in a high-level text format.
- When the project is compiled, the `mlir-tblgen` tool parses `.td` files and automatically writes corresponding `c++` (more on this in [MLIR Compilation](MLIR%20Compilation.md))
- Every operation definition in TableGen consists of a structural schema where you state the inputs, outputs, traits, and documentation for your operation. 

```TableGen
include "mlir/IR/OpBase.td"

def MathX_Dialect : Dialect { 
	let name = "mathx"; 
	let cppNamespace = "::mlir::mathx"; 
	let summary = "An example extension math dialect."; 
}

// Inherit from generic OP class, inheriting properties like summary, description, results, and passing on traits, dialect, and Mnemonic.
class MathX_Op<string mnemonic, list<Trait> traits = []> :
	Op<MathX_Dialect, mnemonic, traits>

// Inherit from the base Op class of MathX
def MathX_ScaleOp : MathX_OP<"scale", [Pure]> {
	let summary = "Scales a tensor by a scalar factor";
	let description = [{...}];
	
	let arguments = (ins
		F32Tensor:\$input_tensor // SSA value/runtime value.
		F32Attr:  \$scale_factor // Attribute.
	)
	
	let results = (outs 
		F32Tensor:\$output_tensor 
	);
	
	//This dictates our custom format.
	let assemblyFormat = "\$input_tensor `,` \$scale_factor     attr-dict `:` type(\$input_tensor) `->`  
	type(\$output_tensor)";
	
}
```
- In custom operation format, the `MathX_ScaleOp` we defined above would look like so in `.mlir`
```
%result = mathx.scale %my_tensor, 2.5 : tensor<10x10xf32> -> tensor<10x10xf32>
```
- Where 2.5 is our `F32Attr`

