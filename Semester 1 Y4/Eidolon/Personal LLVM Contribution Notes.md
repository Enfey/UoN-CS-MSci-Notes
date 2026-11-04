## Issue 1
- For a power of 2:$$d = 2^k, x \ mod \ d = x \ \& \ (d-1)$$
- Subtracting a 1 from a power of two yields a bitmask of $k$ ones that spans from the LSB to represent $d-1$ 
	- E.g., 1000 -> 0111
- We merely preserve the lowest bits of $d$ according to the set bits in $x$
- This is one instruction
- A division on AMDGPU is about 18, as there is no direct division operation; it is a lengthy sequence of reciprocal estimation, scaling, multiplication, etc.
### Landscape
![](Pasted%20image%2020261104033534.png)
- LLVM IR is a target independent, typed representation
- Modulo would look akin to this: `%r = urem i32, %x, $d`
- The middle-end optimises IR for every target
	- A number of passes are involved here; for example , InstCombine is a target-independent canonicalisation pass that simplifies arithmetic, bitwise, comparison, memory, and cast operations without relying on target-specific machine instructions
	- Does this by using LLVM's pattern matching infrastructure
	- Although LLVM IR is target independent, the IR produced by the frontend is still tightly bounded to the machine it ran on e.g., pointer sizes typically `i64` when the target may be 32 bit embedded platform for example
	- The middle-end actually performs our fold at the IR level by using value tracking, a library of helper routines that statically deduce properties of LLVM IR values e.g., whether an integer is known to be non-zero
- The backend turns the optimised IR into one target's instructions
	- The first job of the backend is naturally, instruction selection. 
- Now ideally, we want to preserve the fold we performed in the middle-end (if at all, we may have code that was not touched by InstCombine)
- There are **two** instruction selectors we are bothered about:
	1. **SelectionDAG**
		- This is the older, established instruction-selection framework
		- It converts LLVM IR into a graph of nodes representing operations and their dependencies.
		- This is quite sophisicated and also models value types, chains for ordering side effects of instructions, and other target specific details
		- The broad process is to construct SelectionDAG nodes given LLVM IR instructions, perform DAG-level optimisations, legalise by transforming operations or types that the target cannot directly support e.g., `i48` becomes `i64`, and then match the DAG patterns found against target-specific instruction definitions and emit them. 
	2. **GlobalISel**
		- This is the newer instruction-selection framework![](Pasted%20image%2020261104035124.png)
		- We have the IR translator which builds gMIR from our LLVM IR that comes from our middle-end, which uses target independent opcodes e.g., `G_TRUNC`, `G_CONSTANT` on virtual registers with low-level types (LLTs): `sN` represent a raw scalar bit bundle of size $N$; `<vN x SM>` or `<N x SM>` denote a SIMD vector type defining the element (count `N`) and the low-level scalar type of each element (`sM`)
		- We want to perform our fold in the `PreLegalizerCombiner`, and then if we determine what we are dividing is not a power of two and we can't convert to a bitmask, the legalizer will re-expand `G_UREM` to roughly 18 instructions to be converted to AMDGPU instructions by the instruction selection portion of GlobalISel
		- Combiners are merely peephole optimisers.

### Combines
- A combine has three layers
	1. **Rule**
		- This is written in `TableGen`, LLVM's declarative description language
		- `urem_pow2_to_mask` matches the shape `G_UREM %dst, %x, %y`, and then runs a `C++` condition written inside some following brackets
		- If that condition matches, it runs the `C++` "apply" step
		- At build-time, `llvm-tblgen` compiles every rule into generated C++ matchers (`.inc` files)
		- Rules are grouped: `urem_pow2_to_mask` is in `known_bits_simplification`, which is in `all_combines`and so affects all backends which run through GlobalISel, which means this change affects AMDGPU, AArch64, and RISC-V 
	2. **Bridge**
		- CombinerHelper is a shared library of match and apply functions that rules for combines in GlobalISel defer to.
		- We pass through to this, which is a thin-wrapper around the actual analysis
		- If the pass-through to analysis returns true (for our case) then we simplify the IR and perform the fold as per the rule (which we specify as following the condition which will be determined by the analysis)
	3. **Analysis**
		- `isKnownToBeAPowerOfTwo` answers whether a given register is a power of two
		- gMIR is SSA form, and thus every virtual register has precisely one defining instrction
			- MachineRegisterInfo(MRI) tracks this, along with some other properties e.g., LLT of register.
				`MachineInstr *DefMI = MRI->getVRegDef(Reg);`
		- `GISelValueTracking` is GlobalISel's equivalent of the LLVM IR `ValueTracking` utility
		- It operatres on top of `MRI` and the single-definition nature of gMIR; MRI gives you the raw defining instruction, and some properties, but value tracking is able to tell you what the underlying data/value inside that register looks like statically; known bits analysis.
			- However, it is entirely optional so the pointer may be null for any given virtual/physical register.

Essentially, there are three versions of 'is this a power of two?'. IR valuetracking, SelectionDAG, both had an OrZero flag to return true where this value is also zero. This flag was not present anywhere in GlobalISel, and leads to a missed optimisation.

### The problem
- The rule in our combine in our `prelegalizercombiner` asks for a `G_UREM` whether $d$ is a power of two `isKnownToBeAPowerOfTwo(d)`
- Where $d$ is derived from a logical shift, only where $d$ is defined as $1 << y$ do we return true from this
- This is because shifting any other power of two can push the bit off the top and leave zero
	- For example, 4096 << 20 is 2^32, which is 0 in a 32-bit register.
- Zero is not a power oif  two, and so the fold is rejected, and the legaliser emits the full 18 instruction division, which is subsequently converted the backend equivalent instructions. 
	- $1 << y$ is only accepted because its bit cannot fall off unless $y$ exceeds 32 for $s32$, and a shift that large is already poison. 
- For $urem$ zero does not matter, because division by zero is undefined behaviour; the program is already broken, so why emit a long instruction sequence when whatever the mask produces will be acceptable?


### The fix
- Port SelectionDAG's `OrZero` mode into the GlobalISel analysis portion of the combine in the prelegalizercombine
- The rules are as follows, by which the instruction defines the divisor:
	- `G_CONSTANT` - accept a power of two, a negated power of two if `OrNegative` is set; and zero if `OrZero` is set.
	- `G_SHL`
		- Keep the $1 << y$ shortcut
		- Otherwise if `OrZero` is set or the result is known never to be zero and the value being shifted qualifies, then return `true`
		- We recurse on registers by taking their defining instruction until we reach the base definition of `%d`
		- ....
rest makes sense, lets roll