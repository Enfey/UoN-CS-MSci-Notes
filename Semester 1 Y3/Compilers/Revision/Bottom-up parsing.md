# Paradigm summary
> Begin with leaves, and grow tree toward the root, applying productions in reverse(process is called reduction). Each step, identify a substring of that matches the RHS of some production, build node for LHS and connect to the tree.

### High-level desc
- To build a derivation, add layers of NTs atop the leaves in structure dictated by grammar, and the partially completed portion of the parse tree.
- To extend tree upward, look in frontier for substring matching RHS of some production $A \to \beta$, if it finds $\beta$, with its right end at $k$, can replace $\beta$ with $A$ to create a new frontier. 
### Bottom-up basics
- Start with input setence, apply productions backwards.
- For a given form $\gamma_i$, determine what $y_{i-1}$ must have looked like to produce it
	- What production rule was applied?
	- What prefix of the input was consumed?
- A **handle** is a pair of a production rule, and an index into a **sentential form**
	- $(A \to \beta, k)$ means the first $k$ tokens of $\gamma_i$ form $\beta$, and so $\beta$ can be replaced with $A$ in $y_{i-1}$.
- The replacement is called a reduction, these are applied until one reaches the start symbol/goal symbol(or until a handle cannot be found, cannot build derivation). 
![](Pasted%20image%2020260119131530.png)

Bottom-up parser effectively discovers the steps of a derivation in reverse order. For a derivation: $$Goal = \gamma_0 \to \gamma_1 \to \gamma_2 \to \dots \to \gamma_{n-1} \to \gamma_n = sentence$$
The bottom up parser discovers $\gamma_i \to \gamma_{i+1}$ before it discovers $\gamma_{i-1} \to \gamma_i$. 
### LR(1) parsing
- Informally, means that **L**eft to right scan, **Reverse rightmost derivation**, 1 token of lookahead
- Completes bottom-up parse in $O(n)$ time based on size of derivation
- Algorithm pushes symbols onto a stack, until the top of the stack forms a handle
	- The stack is then reduced by popping the RHS of the handle, and pushing the LHS.
	- Repeat until error, or goal symbol reached.
- $k$ = lookahead symbol.

#### LR(1) tables
- **Action table**
	- Tells whether to shift or reduce a terminal symbol
	- Indicates the action to perform, and the state to move to
	- **One row per state, one column per terminal.** 
		- Ie., whether to shift or reduce, or do nothing in that state.
		- ![](Pasted%20image%2020260119132300.png)
- **Goto table**
	- Tells what state to go to with a **nonterminal** on the stack
	- No input consumed, state just changes
	- **One row per state, one column per nonterminal.**
		- ![](Pasted%20image%2020260119132414.png)


#### LR(1) construction + items
- An **LR(1)** item is a possible configuration the parser may find itself in. $$[A \to \beta \cdot \gamma, a]$$
- Consists of a production $A \to \beta$
- A placeholder $\cdot$, that indicates the position of the parsers stacktop(already have $\beta$ on the stack in this case, need to find $\gamma$
- And the next terminal symbol $a$
- There are 3 configurations:
	1. $[A \to \cdot \beta \gamma, a]$
		Indicates an $A$ would be valid, recognising $\beta$ would be one step toward discovering an $A$. Possible item. 
	2. $[A \to \beta \cdot \gamma, a]$ 
		Recognised $\beta$, next step is to validate $\gamma$. Partially complete item.
	3. $[A \to \beta \gamma \cdot, a]$ 
		Recognised $\beta$ and $\gamma$ in a context where an $A$ followed by an $a$ would be valid. The parser can reduce $\beta \gamma$ to $A$. Complete item.

- The **canonical collection** is a set of sets of items
	- Each set $cc_i \in CC$ represents a different state. 

#### Closure Function
- The **closure** of an item is all other items that item implies
	- e.g. $[𝐺𝑜𝑎𝑙 → \cdot 𝐿𝑖𝑠𝑡, eof]$ implies $[𝐿𝑖𝑠𝑡 → \cdot 𝐿𝑖𝑠𝑡 𝑃𝑎𝑖𝑟, eof]$
	- OR alternatively:
		- The item $[Goal→• List,eof]$ implies both $[List→• List Pair,eof]$ and $[List→• Pair,eof]$, with respect to the **placeholder.** 
- To build the closure of $[A \to \beta \cdot C\delta, a]$, find all productions $C \to \gamma$ and for each symbol $s \in First(\delta a)$, create item $[C \to \cdot \gamma, s]$ 
- take the productions of the NT, for each production, for all s in the first sets of delta, a, create a unique item.

For the parentheses grammar, the initial item is $[Goal→• List,eof]$. Applying closure to that set adds the following items: $$[List → • List Pair,eof], [List → • List Pair,( ], [List → • Pair,eof ], [List → • Pair,( ], [Pair → • ( Pair ),eof ], [Pair → • ( Pair ),(], [Pair → • ( ),eof] [Pair → • ( ),(]$$
- We use this to create $cc_0$ 
#### GoTo function
- The **goto** function takes a set of items $cc_i \in CC$ and a grammar symbol. 
- If the item is $[A \to \beta\cdot x\delta, a]$ and the symbol is $x$:
	- Add the new item $[A \to \beta x \cdot \delta, a]$ to the goto set.
- Once all items in $cc_i$ have been processed for a given grammar symbol, take the closure of each, combine them, and then return. 

#### Construction trace and algorithm.
![](Pasted%20image%2020260119140951.png)
Repeat this process:
1. Select $cc_i \in CC$ that hasn't already been chosen
2. For each grammar symbol called s:
	Compute $cc_j = goto(cc_i, s)$(goto this state on this symbol).
	Compute $cc_j = closure(cc_j)$ 
	Add $cc_j$ to $CC$.


##### Step 1
Compute closure of $s$ to get $cc_0$, can only expand NTs looking at, those are the implied transitions.
![](Pasted%20image%2020260119141331.png)
Add to worklist.
While the worklist is not empty, remove from worklist, for each symbol X, compute goto(cc, X)
if the goto non empty, and not IN CC, add to CC, add to worklist.
##### Step 2
Compute goto
If the item is $[A \to \beta\cdot x\delta, a]$ and the symbol is $x$:
	- Add the new item $[A \to \beta x \cdot \delta, a]$ to the goto set.
![](Pasted%20image%2020260119141454.png)

##### Step 3
Compute goto again
![](Pasted%20image%2020260119141529.png)


##### Step 4
Compute goto again
![](Pasted%20image%2020260119141608.png)



![](Pasted%20image%2020260119150128.png)
