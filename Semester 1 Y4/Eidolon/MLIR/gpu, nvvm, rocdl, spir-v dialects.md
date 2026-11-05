# Overview
- The `gpu` dialect organises gpu related mid-level abstracted operations, types, attributes, dataflow analyses, and transformation rules.
- Naturally, it defines its own domain rules but leverages MLIR's core framework features e.g., debug info tracking discussed in the [MLIR Overview](MLIR%20Overview.md).
- To begin with, we'll focus only on the collective operations. As Eidolon potentially expands, we may extend to matrix ops, execution-model queries and other gpu capabilities.

## `gpu` dialect
### `gpu.shuffle`
- **Short Description**
	- *Shuffles values within a subgroup*
	- The **shuffle** op moves values across lanes in a subgroup. 
	- The `width` argument specifies the number of lanes that participate in the shuffle, and must be uniform across all lanes. 
	- Furthermore, the first `width` lanes of the subgroup must be *active*. 
	- Returns the `shuffleResult` and `true` if the current lane id is smaller than `width`, and an unspecified value and `false` otherwise.
- **Syntax**
```
operation ::= `gpu.shuffle` enum($mode) $value `,` $offset `,` $width attr-dict `:` type($value)
```
- **Further Description**
	- A shuffle permits one lane read a value held by another lane in the same subgroup.
		- There are multiple methods in which this value is chosen, governed by `enum($mode)`
	- A shuffle does not read from shared memory on the SM and instead permit direct reading from the register file where this would normally be forbidden
	- It does not require a barrier between lanes to do so.
- **Formal Description:**
	- Let $N$ be the subgroup size
	- Let $k$ be the ID of the lane executing `gpu.shuffle`
	- $x_{j}$ to be the distinct value operand held by lane $j$, for each lane $j$
	- $s(k)$ be the source lane selected by the shuffle
	- The shuffle result in the simplest case (direct access) is then for lane $k$ is then $y_k = x_{s(k)}$
		$s(k)$ is governed by `enum($mode)`
		For direct access,  we merely specify a lane id and broadcast it to all threads; say we chose $16$ and all threads were participating, all threads would end up possessing the operand described by $x_{16}$ as $s(k) = x_{16}$ for all threads.
	This is technically incorrect however; `gpu.shuffle` returns a validity bit thus the result is a pair $y_k, v_k$ where $y_k$ is the shuffled value for lane $k$ and $v_k$ indicates whether the result is valid according to the ops documented rules.

#### Distinction between subgroup size and width
- Subgroup size $N$ describes the number of lanes in the subgroup, with width $w$ decribing the number of lanes participating in the `shuffle` operation
- The documentation mandates that the width argument specifies the number of lanes that must participate in the shuffle, and must be **uniform across all lanes**: $$\forall i, j \in A, w_i = w_j$$
	Every active lane must agree on the width.
- The first $w$ active lanes are also required to be active. 
- We distinguish:
	- **Active lanes**: lanes participating in the current execution; we call this set $A$
	- **Shuffle-participating lanes**: the first $w$ lanes, according to the document width requirement
	- **Valid-result lanes**: lanes for which the operation reports $v_k = true$ 
- These naturally have overlap, but are not interchangeable, e.g., we may have invalid results as active lanes may exceed $w$; active lanes which become inactive at the moment of the shuffle e.g., lanes 16-31 diverge and do not perform the shuffle.

#### Semantics implications
- `%offset` and `%width` are SSA values and are not necessarily compile-time constants
- This is relevant because the documentation clearly imposes a uniformity requirement on `width`, but the type system does not express that requirement because the values may be determined at runtime and not preserve said uniformity requirement.
- The validator we write cannot assume uniformity then. 
- This is very troublesome for `%width`, because there are possibilities that $1 \leq w \leq N$ will not be true at runtime.
	- Width could be 0 (which i think just means that all participating lanes performing shuffles return false as part of their result), less than 0, or exceed the subgroup size.
	- Nothing says what violating the width requirement does (UB or poison)
- The ODS specification for `gpu.shuffle` attaches `Pure`, which essentially tells the infrastructure that the operation does not have side effects.
	- The op is also `memoryEffectFree()`
	- This has the following implications below...
- The shuffle operation has the precondition that $\{0, ... w-1\} \Subset A$, which can be invalidated by a compiler transformation. 
	- For example, control-flow sinking is a compiler optimisation pass that moves instructions from a basic block $A$ into more narrowly executed successor block (we'll call it $B$)
		- This is usually say, the branches of an $if$ statement, or inside loops.
		- Moves further down the CFG closer to where the result is actually needed.
		- For classic code sinking, there is a dominance requirement, that is not a fundamental requirement for **speculative code sinking**
	- The difference between speculative and regular code sinking is not of massive importance, but the idea is that the code is moved without guaranteed legality, but the profitability tradeoff is worth the move.
	- Now this is an issue for us:
		Picture a CFG:
		![](Pasted%20image%2020261105002905.png)
		Sunk into the else branch, where it has a 90% chance of executing shall we say for lane $k$:
		![](Pasted%20image%2020261105002947.png)
		Suppose only some lanes take the `then` branch
		The shuffle now executes with a different active lane set to prior to the optimisation, usually being a strict subset: $A' \subset A$ meaning there is a chance that $w \gt A'$ , meaning the shuffle would introduce undefined behaviour. 
	- This has actually caused a bug in the past via `-control-flow-sink`
#### Result
- We return a pair of the `shuffleResult`, and `true` if the current lane id is smaller than width $w$, and an unspecified value and false otherwise.
- Unspecified is not the same as poison. 
	- The specification does not promise a particular valid value or result, but this does not mean the actual execution that occurred was invalid. 
	- Suppose the source produces an unspecified value $y_{k} \in \mathbb{Z}_{32}$, this may be entirely valid depending on the target architecture and program requirements, but would still return `false` as part of the tuple. 
	- We do not actually know however, the documentation does not make precise what unspecified values are, which makes lanes that continue executing with the result effectively UB.
	- `Poison` is a value representing deferred UB; it can propagate through subsequent operations and eventually cause undefined behaviour when used in certain ways; it is essentially a black box.

#### Modes
- The interpretation of the `%offset` arguments depends on the selected mode
- Each mode essentially defines how to acquire $s(k)$ 
- Nothing says that offset must be uniform, so different offsets per lane must be legal.
##### XOR
- For lane $k$, returns the value `%0` from lane $k \textasciicircum{}offset$ 
- Every participating lane trades value with exactly one other lane.
- Formally:
	- For $k < w$, the result is $(x[k \oplus  o], true)$ 
- Given that XOR is its own inverse under our circumstances, applying the same XOR shuffle twice restores the original lane values.
###### Semantic Implications
- Suppose $N = 8, w = 6, o = 1$ 
- For lane $5$:
	- $y_5 = 5 \oplus 2 = 7$ 
- $k \lt w$ so $v_5 = true$, but the selected source lane lines outside the range of lanes guaranteed to be active defined by $w$
- This is not necessarily UB, lane 7 may be active. It may also not.
- The source selection rule for `gpu.shuffle` modes, and the activity precondition $\{0, ... w-1\} \Subset A$ do not fit together cleanly.
- The documentation does not further require that $o \lt w$, nor that $o$ is a power of two, which is useful for XOR shuffles as $k \oplus o$ will never exceed $w$ and thus the result will not be unspecified.
- The documentation describes that lanes trade value with exactly one other lane, but where $o = 0$, $k \oplus 0 = k$, every lane selects itself and is redundant.
##### Down
- For lane $k$, returns the value from lane $(k+cst1)$ 
- If $(k+cst1)$ is bigger than or equal to width, the value is poison and `valid` is `false`
- For $k < w$, if $k + o \gt w$, the result is $(UB, false)$, otherwise ($x[k+o], true$ )
- A negative $o$ is not mentioned by the documentation, they may be supported...
- An overflowing i32 as $k+o$ is also not stated.

##### Up
- For lane $k$, returns the value from lane $(k - cst1)$. 
- If $(k-cst1)$ is smaller than $0$, the value is `poison` and `valid` is `false`
- For $k \lt w$, if $k - o < 0$, the result is $(UB, false)$, otherwise $(x[k-o], true)$ 
- A negative $o$ makes $k-o > k$ possibly $\ge w$, and this case is undefined; we are reading from a lane that is not participating in the shuffle, and may not be active.
##### IDX
- Broadcasts the value from lane $0$ to all lanes
- $y_k = x[0]$ for all $k$

##### General validity conditions and specific validity conditions
- $k < w$ is generally the only documented validity condition for all 4 modes.
- Up and Down introduce additional cases where the source lane cannot be less than zero, or greater than width, respectively.
	- These are technically true for both, except only one for each says that Poison is returned, and false.
#### TableGen/ODS
![](Pasted%20image%2020261105014837.png)

#### Lowering
- These are simplified and just for comprehension as i'll just refer to the main branch of LLVM when developing Eidolon. 
##### `nvvm`
- This is defined in `LowerGpuOpsToNVVMOps`:
```mlir
%n = llvm.sub 32, %width
%mask = llvm.lshr -1, %n
%c = llvm.sub %width, 1
%r = nvvm.shfl.sync bfly|down|up|idx %mask, %value, %offset, %c
%v = llvm.extractvalue %r[0]
%ok = llvm.extractvalue %r[1]
```
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