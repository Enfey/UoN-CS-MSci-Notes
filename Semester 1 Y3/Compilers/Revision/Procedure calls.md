# Motivation
- Until now, only considered programs that form single name
- Catch all name for callable unit = procedure

### Compiling with procedures
- Up until the final moment, we can compile procedures as independent units of code
- Internal **Linkage Conventions** define how separate units of code should be combined into a single program. 
	Aligned with the **calling convention** and ABI for specific architecture and OS. 

### ABI
> Application binary interface, set of programming conventions that OS's must ensure applications follow to run under that system

- Contract between code emission and consumption
- Number of constraints with regard to procedures
	- **Parameter passing** - where parameters go(stack or register), order, who pushes them.
	- **Return values**: how many words, where they are placed(register or stack), how caller reads them
	- **Call/return sequence**: exactly what bytes/instructions constitute a call and return, and how are return addresses and links stored
	- **Frame layout/stack discipline**: where locals live, where saved registers live, alignment rules
	- **Register preservation**: which registers callee must preserve vs which registers the caller must preserve.
	- **Unwinding model**
- If follow ABI, modules compiled independently will interoperate.


### Procedure calls
- Central abstraction
- The linkage convention specifies an agreement (between the compiler and OS according to the ABI) on the actions taken to call a procedure/function. 
- Can split stages of a procedure call:
	- **Precall**
	- **Function Prologue**
	- **Function Epilogue**
	- **Postcall code**
### Activation record/Stack frame
> An activation record is a region of storage associated with a single, isolated invocation of a procedure.

- ***Section of memory used to communicate interprocedurally***
- Procedures access their AR frequently, thus, an activation record pointer/frame pointer holds the address of the current AR(frame pointer usually acts as ARP).(can just be an offset from the stack pointer, and stay fixed.
- ![](Pasted%20image%2020260119185408.png)
- Provides space for:
	- Parameters(from call site)
	- Local data area
	- Return values
	- Return address - runtime address where execution should resume when procedure terminates.
	- Saved registers
	- Pointer to caller's ARP, needed to restore caller's environment upon return. 
- The stack pointer points to the top of the stack, and changes frequently during function execution. 

#### Generating ARs
- The compiler must emit code that sets up and tears down a new activation record each time a procedure is called
- Responsibility is divided between the caller and the callee.

#### Generating ARs: Caller actions
- **Precall sequence**
	- Caller **must** <mark style="background: #FFF3A3A6;">calculate</mark> actual <mark style="background: #FFF3A3A6;">parameters</mark> and <mark style="background: #FFF3A3A6;">return address</mark>
		- Caller **must** <mark style="background: #FFF3A3A6;">save registers</mark>, if register based machine
	- Caller <mark style="background: #FFF3A3A6;">**may**</mark> <mark style="background: #FFF3A3A6;">allocate space</mark> for <mark style="background: #FFF3A3A6;">local variables</mark>
	- Transfer control to callee.
- **Postcall sequence**
	- Caller **stores return value** in appropriate location

#### Generating ARs: Callee actions
- **Prologue**
	- Allocate memory for local variables if not done by caller(some of which may have been passed by register)
- **Epilogue**
	- **Evaluate** and **store return value** in specified location
	- Deallocate memory/**unwind stack**
	- **Transfer control back** to caller.

### Passing parameters
#### Pass by value
- Used for small values, copied
- Value is evaluated by caller, and stored at appropriate location relative to AR
- Compiler must ensure, generated code, does not modify source values
	- Caller places copy of value, not address, in AR, and callee must only modify that copy.
#### Pass by reference
- Callee receives the address of the value used as a parameter
- Caller **generates code** to calculate address and store in relevant place in AR/register
- Generated code in callee must respect reference semantics e.g., dereference to modify.


### Linking
- Linking convetion, agreed method for where parameters should be found, who stores return address and where, etc.
- Essentially derived from the ABI, determines AR layout.
- Important for interoperability
- TAM
	- Parameters loaded onto stack
	- CALL creats new stack frame
		- LB updates to point to base of new stack frame
		- Base of caller's stack frame stored in 1[LB]
		- Return address stored in 2[LB]
	- RETURN instruction resets LB to stack frame, pops all local values from stack, and loads return address on top. 

