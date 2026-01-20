# Paradigm summary
> <mark style="background: #FFF3A3A6;">Begin with leaves</mark>, and<mark style="background: #FFF3A3A6;"> grow tree toward the root</mark>, <mark style="background: #FFF3A3A6;">applying productions</mark> in<mark style="background: #FFF3A3A6;"> reverse</mark>(process is called reduction).<mark style="background: #FFF3A3A6;"> Each step</mark>, <mark style="background: #FFF3A3A6;">identify</mark> a <mark style="background: #FFF3A3A6;">substring</mark> of that <mark style="background: #FFF3A3A6;">matches</mark> the <mark style="background: #FFF3A3A6;">RHS</mark> of <mark style="background: #FFF3A3A6;">some production</mark>, <mark style="background: #FFF3A3A6;">build node </mark>for <mark style="background: #FFF3A3A6;">LHS</mark> and connect to the tree.

### High-level desc
- To build a derivation, <mark style="background: #FFF3A3A6;">add layers </mark>of <mark style="background: #FFF3A3A6;">NTs</mark> <mark style="background: #FFF3A3A6;">atop the leaves</mark> in structure dictated by grammar, and the partially completed portion of the parse tree.
- To <mark style="background: #FFF3A3A6;">extend tree upward</mark>, look in <mark style="background: #FFF3A3A6;">frontier</mark> for <mark style="background: #FFF3A3A6;">substring</mark> <mark style="background: #FFF3A3A6;">matching </mark><mark style="background: #FFF3A3A6;">RHS</mark> of some <mark style="background: #FFF3A3A6;">production</mark> $A \to \beta$, if it<mark style="background: #FFF3A3A6;"> finds</mark> $\beta$, with its right end at $k$, can replace $\beta$ with $A$ to create a new frontier. 
### Bottom-up basics
- <mark style="background: #FFF3A3A6;">Start with input setence</mark>,<mark style="background: #FFF3A3A6;"> apply productions</mark> <mark style="background: #FFF3A3A6;">backwards.</mark>
- For a given form $\gamma_i$, determine what $y_{i-1}$ must have looked like to produce it
	- <mark style="background: #FFF3A3A6;">What production rule was applied?</mark>
	- <mark style="background: #FFF3A3A6;">What prefix of the input was consumed?</mark>
- A **handle** is a pair of a production rule, and an index into a **sentential form**
	- $(A \to \beta, k)$ means the first $k$ tokens of $\gamma_i$ form $\beta$, and so $\beta$ can be replaced with $A$ in $y_{i-1}$.
- The <mark style="background: #FFF3A3A6;">replacement</mark> is <mark style="background: #FFF3A3A6;">called</mark> a <mark style="background: #FFF3A3A6;">reduction</mark>, these are<mark style="background: #FFF3A3A6;"> applied</mark> until one <mark style="background: #FFF3A3A6;">reach</mark>es the <mark style="background: #FFF3A3A6;">start symbol</mark>/goal symbol(<mark style="background: #FFF3A3A6;">or until a handle cannot be found,</mark> cannot build derivation). 
![](Pasted%20image%2020260119131530.png)

<mark style="background: #FFF3A3A6;">Bottom-up parser effectively discovers the steps of a derivation in </mark><mark style="background: #FFF3A3A6;">reverse order</mark>. For a derivation: $$Goal = \gamma_0 \to \gamma_1 \to \gamma_2 \to \dots \to \gamma_{n-1} \to \gamma_n = sentence$$
The bottom up parser discovers $\gamma_i \to \gamma_{i+1}$ before it discovers $\gamma_{i-1} \to \gamma_i$. 
### LR(1) parsing
- Informally, means that **L**eft to right scan, **Reverse rightmost derivation**, 1 token of lookahead
- Completes bottom-up parse in $O(n)$ time <mark style="background: #FFF3A3A6;">based on size of derivation</mark>
- Algorithm <mark style="background: #FFF3A3A6;">pushes symbols onto a stack</mark>, until the <mark style="background: #FFF3A3A6;">top of the stack </mark><mark style="background: #FFF3A3A6;">forms a handle</mark>
	- The <mark style="background: #FFF3A3A6;">stack is then reduced</mark> by <mark style="background: #FFF3A3A6;">popping the RHS of the handle, </mark>and pushing the LHS.
	-<mark style="background: #FFF3A3A6;"> Repeat until error</mark>, or<mark style="background: #FFF3A3A6;"> goal symbol reached.</mark>
- $k$ = lookahead symbol.

#### LR(1) tables
- **Action table**
	- <mark style="background: #FFF3A3A6;">Tells whether</mark> to <mark style="background: #FFF3A3A6;">shift</mark> or <mark style="background: #FFF3A3A6;">reduce</mark> a <mark style="background: #FFF3A3A6;">terminal symbol</mark>
	- <mark style="background: #FFF3A3A6;">Indicates</mark> the <mark style="background: #FFF3A3A6;">action to perform,</mark> and the<mark style="background: #FFF3A3A6;"> state to move to</mark>
	- **One row per state, one column per terminal.** 
		- Ie., <mark style="background: #FFF3A3A6;">whether to shift or reduce</mark>, <mark style="background: #FFF3A3A6;">or do nothing in that state.</mark>
		- ![](Pasted%20image%2020260119132300.png)
- **Goto table**
	-<mark style="background: #FFF3A3A6;"> Tells what state to go to</mark> with a **nonterminal** <mark style="background: #FFF3A3A6;">on the stack</mark>
	- <mark style="background: #FFF3A3A6;">No input consumed</mark><mark style="background: #FFF3A3A6;">, state just changes</mark>
	- **One row per state, one column per nonterminal.**
		- ![](Pasted%20image%2020260119132414.png)
<mark style="background: #FFF3A3A6;">ACTION = TERMINAL, GOTO = NONTERMINAL, GOTO JUST HAS STATES, ACTION HAS SHIFT+REDUCE AND STATES.</mark>

#### LR(1) construction + items
- An **LR(1)** item is a <mark style="background: #FFF3A3A6;">possible configuration</mark> the<mark style="background: #FFF3A3A6;"> parser</mark> may<mark style="background: #FFF3A3A6;"> find itself in. </mark>$$[A \to \beta \cdot \gamma, a]$$
- Consists of a production $A \to \beta$
- A placeholder $\cdot$, that <mark style="background: #FFF3A3A6;">indicates the position of the parsers stacktop</mark>(already have $\beta$ on the stack in this case, need to find $\gamma$
- And the next terminal symbol $a$
- There are 3 configurations:
	1. $[A \to \cdot \beta \gamma, a]$
		<mark style="background: #FFF3A3A6;">Indicates</mark> an $A$ would be valid, recognising $\beta$ would be one step toward discovering an $A$. Possible item. 
	2. $[A \to \beta \cdot \gamma, a]$ 
		Recognised $\beta$, next step is to validate $\gamma$. Partially complete item.
	3. $[A \to \beta \gamma \cdot, a]$ 
		Recognised $\beta$ and $\gamma$ in a context where an $A$ <mark style="background: #FFF3A3A6;">**followed by an $a$</mark>** would be valid. The parser can reduce $\beta \gamma$ to $A$. Complete item.

- The **canonical collection** is a s<mark style="background: #FFF3A3A6;">et of sets of items</mark>
	- Each set $cc_i \in CC$ <mark style="background: #FFF3A3A6;">represents a different state</mark>. 

#### Closure Function
- The **closure** of an item is <mark style="background: #FFF3A3A6;">all other items that item implies</mark>
	- e.g. $[𝐺𝑜𝑎𝑙 → \cdot 𝐿𝑖𝑠𝑡, eof]$ implies $[𝐿𝑖𝑠𝑡 → \cdot 𝐿𝑖𝑠𝑡 𝑃𝑎𝑖𝑟, eof]$
	- OR alternatively:
		- The item $[Goal→• List,eof]$ implies both $[List→• List Pair,eof]$ and $[List→• Pair,eof]$, with respect to the **placeholder.** 
- <mark style="background: #FFF3A3A6;">To build the closure </mark>of $[A \to \beta \cdot C\delta, a]$, find all productions $C \to \gamma$ and for each symbol $s \in First(\delta a)$, create item $[C \to \cdot \gamma, s]$ 
- <mark style="background: #FFF3A3A6;">take the productions of the NT</mark>, for<mark style="background: #FFF3A3A6;"> each production,</mark> f<mark style="background: #FFF3A3A6;">or all s in the first sets of delta,</mark> a, <mark style="background: #FFF3A3A6;">create a unique item.</mark>

For the parentheses grammar, the initial item is $[Goal→• List,eof]$. Applying closure to that set adds the following items: $$[List → • List Pair,eof], [List → • List Pair,( ], [List → • Pair,eof ], [List → • Pair,( ], [Pair → • ( Pair ),eof ], [Pair → • ( Pair ),(], [Pair → • ( ),eof] [Pair → • ( ),(]$$
- We use this to <mark style="background: #FFF3A3A6;">create </mark>$cc_0$ 
#### GoTo function
- The **goto** function <mark style="background: #FFF3A3A6;">takes a set of items</mark> $cc_i \in CC$ and <mark style="background: #FFF3A3A6;">a grammar symbol. </mark>
- If the item is $[A \to \beta\cdot x\delta, a]$ and the symbol is $x$:
	- <mark style="background: #FFF3A3A6;">Add the new item</mark> $[A \to \beta x \cdot \delta, a]$ to <mark style="background: #FFF3A3A6;">the goto set.</mark>
- <mark style="background: #FFF3A3A6;">Once all items in </mark>$cc_i$ have been <mark style="background: #FFF3A3A6;">processed </mark>for a<mark style="background: #FFF3A3A6;"> given grammar </mark>symbol, <mark style="background: #FFF3A3A6;">take the closure of each</mark>, <mark style="background: #FFF3A3A6;">combine them,</mark> and <mark style="background: #FFF3A3A6;">then return. </mark>

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
