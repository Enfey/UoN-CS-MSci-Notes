## MLIR
- MLIR is a compiler infrastructure part of the LLVM project  defining many IRs and representations that are called ***dialects***
- These dialects share common structure, tooling, and transformation machinery and can easily be extended.
	- They each represent different levels of abstraction, structure, and semantics.
- MLIR was conceived due to LLVM being too low-level; it describes a generic register machine with integers and floats of fixed widths.
- The issue with LLVM IR is that it was a one-size fits all solution for many different programming languages with different semantics, that all languages targeting LLVM would have to accept that LLVM IR cannot represent those semantics that need to be checked or optimised in language specific ways.
	- For example Rust's borrow checker needs to know that `r` holds an exclusive mutable borrow of `v` for some certain span of code/execution, so the following must be rejected:
```rust
let r = &mut v;
v.push(1); // Error: v is mutably borrowed.
r.push(2);
```

- In LLVM IR there is no notion of lifetimes, borrowing, moves like there are in the Rust frontend, so Rust cannot do borrow checking on LLVM IR. 
	- So it instead defined a control-flow graph IR that still knows about borrows, drops, and Rust types called MIR.
- There are many similar tales of languages bolting their own IR in front of LLVM IR with a subsequent lowering to LLVM IR to ensure certain properties of the language are enforced, as it is difficult to get information about a program after it has been lowered and 'lost'.
- LLVM did try to simply annotate its IR, but its passes mandate that any metadata that said passes do not understand may be dropped, so correctness cannot depend on it.
- These languages kept arriving at similar ideas; an IR with the analysable structure of a lower-level IR e.g., SSA, CFG but with the semantics of the language built in.
	- Each language/team then had to write the same supporting machinery:
		- Lowering, verifiers, parsers, optimisations, test tooling
	- And none of these IRs could share/interoperate.

## The problem that MLIR solves
- Let us have an example:
	- We build a small language for operations on arrays and matrices.
	- We construct a `transpose()` operation; we know that transposing something twice gives us back the original, and we'd like the compiler to exploit this
	- Naturally, LLVM IR and its respective optimiser and machine code-gen come for free, but transpose becomes a pair of loops that copy numbers from one chunk of memory to another. 
	- A double transpose in this case `transpose(transpose(x))` is shuffling memory redundantly.
	- Prior to MLIR one would have to construct an IR in front of LLVM IR with, tracking of source line numbers say.
	- MLIR permits us to declare our operation and its semantics with a few lines of description, and allows us to write rewrite rules e.g., if a `transpose` takes its input from another `transpose`, replace it with its original input. 
		- MLIR's rewriting engine simply finds every place that matches and applies it. 
	- You also specify that `mylang.transpose` becomes MLIRs standard `linalg.transpose`.
		- MLIR's existing infrastructure then can lowering it to loops, then to LLVM IR, then to machine code.
		- Can also use its parser, existing lowering infrastructure, passes, source location tracking etc.
			- Just specify own dialect, optimisations, lowering rules, replacement rules etc.
- In MLIR, every operation from every dialect has the same basic 'shape': a name, some inputs, some outputs, some constant settings, and optionally a block of nested code inside it. 
	- Tools that operate on MLIR understand this generalised structure, and more importantly, semantics
	- MLIR has more facilities for declaring semantics so that shared infrastructure does not degrade any guarantees that your source language mandates.


### Worked example - Matrix Transposition 
```MLIR
%t = "mylang.transpose"(%x) : (tensor<4x8xf32>) -> tensor<8x4xf32> loc("model.my":3:9)
```
- The result: `%t` an SSA value
- `mylang` = dialect prefix
- `tranpose` = operation name
- `(%x)` = operand
- followed by the types of the operand and the result type
- The location specifier denotes where it came from in the source dialect definition, and every op carries one that points to its original definition.
- Once an op has been concretely defined, it can also print in a custom form:
```MLIR
%t = mylang.transpose %x : tensor<4x8xf32> -> tensor <8x4xf32>
```

- Naturally MLIR has no idea what transpose does; in MLIR there are no default instructions; an MLIR operation is like an opaque function to MLIR
- If we switch on `--allow-unregistered-dialect` MLIR parses this without complaint.