# Code shape
> Encapsulates all the design choices, large and small, that the compiler author makes regarding the representation of computation, regardless in both intermediate representation and binary representation. 

Although; ian defines **code shape** to be the general set of rules and heuristics for carrying out the initial pass of code generation(which gets things roughly right, with further phases optimising).

Sits between semantics+optimisation - code shape rules and heuristics ensure structurally reasonable, usually unoptimised code.

The shape of the code constrains optimisations. 

## Accessing data
- Addresses should be kept abstract until the last moment
	- Sections of code may be rearranged or optimised out
- Where possible, all symbols should have relative addresses based on some base e.g., base of current stack frame. 

### Values in memory
- The code shape follows a 2 step pattern. 
- Code for accessing memory requires:
	1. Generating code to calculate the address
	2. Generating code to load data from the calculated address. 
- Symbol table should record **storage location** and **size**.

### Array entries
- Arrays are stored contiguously, so address of array element can be computed from base address and index. 
	![](Pasted%20image%2020260119174648.png)
	1. Generate code to get the index
	2. Multiply index by the element size
	3. Add to base address
	4. Load from computed address
- If using **row-major** order in multi-dimensional arrays, move through one row, before moving to the next
- If using **column-major**m iterate through columns, before moving to next. 
- Address calculation depends on which ordering scheme is used.

### Binary operations
- Generating code for a binary operation is as follows:
	1. Generate code to **compute first operand**
	2. Generate code to **compute second operand**
	3. Generate code to **compute the result**(for TAM, leave on stack)
- Optimisation may permit us to skip some of these steps e.g., register allocation. 
#### Binary Operations: Logical operations
- Require special handling; often **short circuit** and stop evaluate early in certain conditions
	$a \&\& b$, if $a$  false, then $b$ should not be evaluated.
	$a||b$, if $a$ true, then don't need to evaluate $b$ 
- Generated code should respect this
- Not an optimisation, short circuited values are not evaluated. 
- ![](Pasted%20image%2020260119181321.png)


### Control flow
- Generally, want to 
	1. Generate code for a **basic block**
	2. Identify all the possible destinations that can follow the block
	3. Generate jumps/branches/fall-through, with appropriate conditions. 
- CFG can make easy, but there are some simple patterns. 


#### Conditionals
- In linear code, we need to jump over the block not ued
- Lable the first instruction instruction of true block, first instruction of false block, and next instruction after false block. 
- ![](Pasted%20image%2020260119181537.png)
- Imagine lbl1 has jump to lbl3

#### Loops
- After iteration, need to just back to start of loop
- Generate label for next instruction after end of loop body
- Generate label for start of loop
- ![](Pasted%20image%2020260119181629.png)

#### Switch/case
- Case statements are a case of conditional execution
- Label first isnstruction of each test case
- Generate code to test input and branch to appropriate section
- Larger statements may generate a lookup table or a binary search to find the correct destination. 

### Procedures
#### Linkage
- The method for generating procedure calls, is aligned with the **calling convention** and ABI for specific architecture and OS. 
- The shape of the code must reflect this agreement. Discussed more next lecture. 
- 4 components; which vary depending on toolchain
	1. **Precall**
		- Generate code to evaluate actual parameters
		- Save registers
		- Jump to procedure
	2. **Function prologue**
		- Generate code to allocate automatic/stack-local variables
	3. **Function epilogue**
		- Unwind stack
		- Store return value in appropriate place
	4. **Postcall**
		- Clear up any remaining memory
		- Proceed with caller code. 