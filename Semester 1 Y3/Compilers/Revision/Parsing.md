# Paarsing
> Second stage of compiler front end; works with tokens produced by scanner to derive syntactic structure for the program, fitting the words into a grammatical model of the source language.

Formal mechanism for specifying the syntax of the source language is via a **Context Free Grammar**. 

For a stream of words $s$ and a grammar $G$ we say that the process of building a constructive proof that $s$ can be derived in $G$ is called parsing.


## Grammars - CFG
- Formalism, define language, more general than regular expressions - Context-free languages
- Language is described as a series of productions
- Terminals are alphabet symbols
- Non-terminals are the internal symbols, $N \cap T = \emptyset$
- **Parsing** is the process of determining if given input strong belongs to $L(G)$ 
	Random notes
	RHS of production may be empty
	Immediately left recursive
	Immediately right recursive
	Recursion may be indirect
	Non--terminals, like variables e.g., digit in BNF

### Derivations
- Process, constructing proof that a word can be resolved in the grammar.
- $w \in T^*$, $w \in L(G)$.
- How can we check?
	1. Start with start symbol $S$
	2. Apply productions, replace NT with matching productions
	3. Continue applying until string is entirely terminals
	4. If final string matches $w$, then $w \in L(G)$ 
- Replace leftmost non-terminal at each step = **leftmost derivation**
- Replace rightmost non-terminal at each step = **rightmost derivation**
- A grammar is **ambiguous** if there are several possible derivations of the same word. 
- A string $a \in (N \cup T)^*$ such that $S \xRightarrow{*}_G \alpha$ is called a sentential form. The language of a grammar, $L(G) \Subset T*$ consists of all terminal sentential forms: 
$$ L(G) = \{w \in T^* \ |\ S \xRightarrow{*}_G w
\}$$
- Basically if it a terminal string can be derived in zero or more steps, its sentential.


### Left recursion + Transforming a Grammar for Top Down Parsing
- A production rule like $S \to S \underline{+} E$ is **left recursive**
- Parsers which produce **leftmost derivations** cannot handle left recursion, infinite recursion, will not generate new leading symbol to advance search/will have multiple productions with same start, cannot choose in LL(1),
- Solve by introducing a new symbol to the left of $S$ 

#### Left Factoring
- When two alternatives share a common prefix, must factor the grammar; a single lookahead not enough to decide which alternative to choose.
- Given productions of the form $A → αβ₁ | αβ₂ | αβ₃$ where $α$ is the **common prefix** and $β_i$ are the distinct suffixes.
- Left factor them into:
	- $A → a  \ A'$ 
	- $A' \to β₁ | β₂ | β₃$ 
- TAKE OUT THE COMMON PREFIX, then make a separate rule for the distinct part of each thing in a production
- Removes common prefixes from productions

#### Direct Left Recursion Elimination
- For all productions of the form: $$A\to A a_{1} \ | \ A a_{2} \ | \dots \ | \ \beta_{1} \ | \beta_{2} \ | \ $$
	Where $a_{i} \neq \epsilon$
	Where $\beta_{i}$ does not start with $A$
- Replace these with with two sets of productions:
	1. $A \to \beta_1 A' \ | \ \beta_2 A'$
	2. $A' \to a_{1} A' \ | \ a_{n}A' \ | \ \epsilon$ 

$Expression \to Expression + Expression \ | \ Integer \ | \ String$ 

Could be rewritten to avoid recursion as
$Expression  \to Integer \ Expression'  \ | \ String  \ Expression'$
$Expression' \to +Expression  \ Expression'  \ |  \  \epsilon$ 

##### Questions
In book


## Parsing strategies
Goal of parsing, construct a parse tree
- Inner nodes = NTs
- Leaves are terminals representing the input
- Shape of the tree shows which rules were applied in which order
Can be constructed **top-down** or **bottom-up**
### Bottom-up
Covered next lecture

### Top-Down
- Begin with root, and grow tree down toward leaves, selecting nonterminal recursively on lower fringe of the tree, extending it with RHS of production that that rewrites the NT. 
- Continue until input stream exhausted or sentential form yielded. If exhaused, may have selected wrong production, may need to backtrack. Can construct parser and grammar in a way that is backtrack free via left factoring, complete elimination of left recursion and use of a lookahead token to decide which production to take. 
#### LL(1) parsing
- Top down look-ahead parsing algorithm requiring a single character of lookahead
- Parses a grammar in $O(n)$ time relative to the derivation length, no backtracking
- To build an LL(1) parser
	1. Compute the **first** and **follow** sets for each symbol
	2. Compute the **start** set for each production
	3. Use sets to build a table to show which rule to apply for each NT + input symbol combination. 
##### First sets
- $First(\alpha)$ is the set of terminals that can appear at the the start of a sentence derived by $\alpha$ 
- If $\alpha$ is terminal, $\epsilon$ or $eof$ then $First(\alpha)$ has exactly one member, $\alpha$.
- For a production $A \to \beta C$ 
	- If $\beta$ is a terminal, add $\beta$ directly to $First(\alpha)$ 
	- If $\beta$ is a nonterminal, add $First(\beta)$ to $First(\alpha)$ 
	- If $\beta$ can derive the empty string, add $First(C)$ too.

##### Follow sets
- $Follow(A)$ is the set of terminals that can appear immediately after parsing an $A$.
- For a production $C \to \beta A \delta$ 
	- Add $First(\delta)$ to $Follow(A$)
	- If $\delta$ can derive the empty string, add $Follow(\delta)$ to $Follow(A)$ too.
- For a production $C \to \beta A$
	- Add $Follow(C)$ to $Follow(A)$
		- LAST NONTERMINALS GET THE LHS OF PRODUCTION ADDED.
- INIT START SYMBOL WITH EOF.
##### Start sets
- $Start (A \to B \ C)$ is the set of terminals that indicate that the rule $A \to B \ C$ can be applied.
- If $A \to B \ C$ 
	- Add $First(B)$ to $Start(A \to B \ C)$
	- If $B$ can derive the empty string, also add $First(C)$
	- If the whole RHS can derive the empty string, add $Follow(A)$
- Each alternative for a non-terminal has a **separate start set**.
- ADD FIRSTS, ADD CONSECTUTIVE FIRST IF DERIVE EMPTY, IF ALL DERIVE EMPTY, THEN ADD FOLLOW OF THE NT PRODUCTION.


#### Parse table/function
- Parsing can be defined either as table lookup or a function.
- To apply a production $A \to B \ C$, the top symbol of the parse stack must be $A$ and the next token must belong to $Start(A \ \to B \ C)$ 
- A terminal on top of the stack matches the same terminal in the input, and consumes it. 
- Have skeleton parser, that uses stack to manage unmatched portion of parse tree's fringe. 
- Parser initialises stack with $eof$ and $S$ 
- Repeatedly examine symbol at top of stack ($focus$) and the current input. 
- If the focus is terminal, matches next word, both are consumed
- If the focus is NT, consult parse table, determine the correct production according to the Start set. 



A grammar is considered **backtrack-free** if, for any nonterminal A with multiple alternative right-hand sides, the FIRST+ sets of those alternatives are pairwise disjoint (all disjoint). This is known as an **LL(1) grammar.**

For every terminal $w \in FIRST^+$ $(A \to \beta)$, the table entry $Table[A, w]$ is set to the production $A \to \beta$. 
