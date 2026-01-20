# Paarsing
> <mark style="background: #FFF3A3A6;">Second stage</mark> of <mark style="background: #FFF3A3A6;">compiler front end;</mark> <mark style="background: #FFF3A3A6;">works</mark> with <mark style="background: #FFF3A3A6;">tokens</mark> <mark style="background: #FFF3A3A6;">produced</mark> <mark style="background: #FFF3A3A6;">by scanner</mark> to<mark style="background: #FFF3A3A6;"> derive syntactic structure </mark>for the program, fitting the words into a grammatical model of the source language.

Formal mechanism for specifying the syntax of the source language is via a **Context Free Grammar**. 

For a stream of words $s$ and a grammar $G$ we say that the process of building a constructive proof that $s$ can be derived in $G$ is called parsing.


## Grammars - CFG
- Formalism, define language, more general than regular expressions - Context-free languages
- Language is described as a series of productions
- Terminals are alphabet symbols
- Non-terminals are the internal symbols, $N \cap T = \emptyset$
- **Parsing** is the <mark style="background: #FFF3A3A6;">process </mark>of <mark style="background: #FFF3A3A6;">determining</mark> if <mark style="background: #FFF3A3A6;">given input</mark> strong <mark style="background: #FFF3A3A6;">belongs</mark> to $L(G)$ 
	Random notes
	<mark style="background: #FFF3A3A6;">RHS of production may be empty</mark>
	<mark style="background: #FFF3A3A6;">Immediately left recursive</mark>
	<mark style="background: #FFF3A3A6;">Immediately right recursive</mark>
	<mark style="background: #FFF3A3A6;">Recursion may be indirect</mark>
	Non--terminals, like variables e.g., digit in BNF

### Derivations
- <mark style="background: #FFF3A3A6;">Process</mark>, <mark style="background: #FFF3A3A6;">constructing proof </mark>that a <mark style="background: #FFF3A3A6;">word can be resolved</mark> in the grammar.
- $w \in T^*$, $w \in L(G)$.
- How can we check?
	1. <mark style="background: #FFF3A3A6;">Start with start symbol</mark> $S$
	2. A<mark style="background: #FFF3A3A6;">pply productions</mark>, <mark style="background: #FFF3A3A6;">replace NT</mark> with matching productions
	3. <mark style="background: #FFF3A3A6;">Continue applying</mark> until<mark style="background: #FFF3A3A6;"> string is entirely terminals</mark>
	4. If final string matches $w$, then $w \in L(G)$ 
- <mark style="background: #FFF3A3A6;">Replace leftmost</mark> non-terminal at each step = **leftmost derivation**
- <mark style="background: #FFF3A3A6;">Replace rightmost </mark>non-terminal at each step = **rightmost derivation**
- A grammar is **ambiguous** if there are<mark style="background: #FFF3A3A6;"> several possible derivations of the same word. </mark>
- A string $a \in (N \cup T)^*$ such that $S \xRightarrow{*}_G \alpha$ is called a sentential form. The language of a grammar, $L(G) \Subset T*$ consists of all terminal sentential forms: 
$$ L(G) = \{w \in T^* \ |\ S \xRightarrow{*}_G w
\}$$
- Basically <mark style="background: #FFF3A3A6;">if it a terminal string can be derived in zero or more steps, its sentential.</mark>


### Left recursion + Transforming a Grammar for Top Down Parsing
- A production rule like $S \to S \underline{+} E$ is **left recursive**
- Parsers which produce **leftmost derivations** cannot handle left recursion, infinite recursion, will not generate new leading symbol to advance search/will have multiple productions with same start, cannot choose in LL(1),
- Solve by introducing a new symbol to the left of $S$ 

#### Left Factoring
- When <mark style="background: #FFF3A3A6;">two alternatives</mark> share a<mark style="background: #FFF3A3A6;"> common prefix</mark>, must <mark style="background: #FFF3A3A6;">factor the grammar;</mark> a <mark style="background: #FFF3A3A6;">single lookahead not enough</mark> to <mark style="background: #FFF3A3A6;">decide which alternative to choose.</mark>
- Given productions of the form $A → αβ₁ | αβ₂ | αβ₃$ where $α$ is the **common prefix** and $β_i$ are the distinct suffixes.
- <mark style="background: #FFF3A3A6;">Left factor them into:</mark>
	- $A → a  \ A'$ 
	- $A' \to β₁ | β₂ | β₃$ 
- <mark style="background: #FFF3A3A6;">TAKE OUT THE COMMON PREFIX</mark>, then <mark style="background: #FFF3A3A6;">make a separate rule</mark> for the<mark style="background: #FFF3A3A6;"> distinct part</mark> of each thing in a production
- <mark style="background: #FFF3A3A6;">Removes common prefixes</mark> from productions

#### Direct Left Recursion Elimination
- <mark style="background: #FFF3A3A6;">For all productions of the form:</mark> $$A\to A a_{1} \ | \ A a_{2} \ | \dots \ | \ \beta_{1} \ | \beta_{2} \ | \ $$
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
<mark style="background: #FFF3A3A6;">Goal of parsing</mark>, <mark style="background: #FFF3A3A6;">construct</mark> a <mark style="background: #FFF3A3A6;">parse tree</mark>
- Inner nodes = NTs
- Leaves are terminals representing the input
- Shape of the tree shows which rules were applied in which order
Can be constructed **top-down** or **bottom-up**
### Bottom-up
Covered next lecture

### Top-Down
- <mark style="background: #FFF3A3A6;">Begin with root</mark>, and <mark style="background: #FFF3A3A6;">grow tree down toward leaves</mark>,<mark style="background: #FFF3A3A6;"> selecting nonterminal</mark> recursively on lower fringe of the tree, <mark style="background: #FFF3A3A6;">extending it with RHS of production</mark> that that <mark style="background: #FFF3A3A6;">rewrites the NT. </mark>
-<mark style="background: #FFF3A3A6;"> Continue until input stream exhausted</mark> or <mark style="background: #FFF3A3A6;">sentential form yielded.</mark> <mark style="background: #FFF3A3A6;">If exhaused,</mark> may have s<mark style="background: #FFF3A3A6;">elected wrong production,</mark> may need to <mark style="background: #FFF3A3A6;">backtrack</mark>. Can construct parser and grammar in a way that is backtrack free via left factoring, complete elimination of left recursion and use of a lookahead token to decide which production to take. 
#### LL(1) parsing
- <mark style="background: #FFF3A3A6;">Top down look-ahead parsing algorithm requiring a single character of lookahead</mark>
- <mark style="background: #FFF3A3A6;">Parses</mark> a grammar <mark style="background: #FFF3A3A6;">in</mark> $O(n)$ time <mark style="background: #FFF3A3A6;">relative</mark> to the <mark style="background: #FFF3A3A6;">derivation length</mark>, no backtracking
- To build an LL(1) parser
	1. <mark style="background: #FFF3A3A6;">Compute</mark> the **first** and **follow** sets for each symbol
	2. <mark style="background: #FFF3A3A6;">Compute</mark> the **start** set for each production
	3. <mark style="background: #FFF3A3A6;">Use sets</mark> to<mark style="background: #FFF3A3A6;"> build a table</mark> to<mark style="background: #FFF3A3A6;"> show which rule</mark> to a<mark style="background: #FFF3A3A6;">pply for each NT</mark> + input symbol combination. 
##### First sets
- $First(\alpha)$ is the <mark style="background: #FFF3A3A6;">set of terminals</mark> that can <mark style="background: #FFF3A3A6;">appear</mark> at the the <mark style="background: #FFF3A3A6;">start </mark>of a <mark style="background: #FFF3A3A6;">sentence</mark> <mark style="background: #FFF3A3A6;">derived</mark> <mark style="background: #FFF3A3A6;">by</mark> $\alpha$ 
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
- <mark style="background: #FFF3A3A6;">ADD FIRSTS, ADD CONSECTUTIVE FIRST IF DERIVE EMPTY, IF ALL DERIVE EMPTY, THEN ADD FOLLOW OF THE NT PRODUCTION.</mark>


#### Parse table/function
- <mark style="background: #FFF3A3A6;">Parsing</mark> can be<mark style="background: #FFF3A3A6;"> defined</mark> <mark style="background: #FFF3A3A6;">either</mark> as <mark style="background: #FFF3A3A6;">table lookup</mark> or a <mark style="background: #FFF3A3A6;">function.</mark>
- <mark style="background: #FFF3A3A6;">To apply a production</mark> $A \to B \ C$, the <mark style="background: #FFF3A3A6;">top symbol </mark>of the <mark style="background: #FFF3A3A6;">parse stack</mark> must be $A$ and the <mark style="background: #FFF3A3A6;">next token must belong to </mark>$Start(A \ \to B \ C)$ 
- <mark style="background: #FFF3A3A6;">A terminal</mark> <mark style="background: #FFF3A3A6;">on top of the stack</mark> <mark style="background: #FFF3A3A6;">matches the same terminal</mark> in the<mark style="background: #FFF3A3A6;"> input,</mark> and <mark style="background: #FFF3A3A6;">consumes it. </mark>
- Have<mark style="background: #FFF3A3A6;"> skeleton parser</mark>, that <mark style="background: #FFF3A3A6;">uses stack </mark>to <mark style="background: #FFF3A3A6;">manage unmatched portion </mark>of <mark style="background: #FFF3A3A6;">parse tree's fringe</mark>. 
- Parser<mark style="background: #FFF3A3A6;"> initialises stack with</mark> $eof$ and $S$ 
- <mark style="background: #FFF3A3A6;">Repeatedly examine symbol at top of stack </mark>($focus$) and the <mark style="background: #FFF3A3A6;">current input. </mark>
- <mark style="background: #FFF3A3A6;">If the focus is terminal</mark>, matches next word, <mark style="background: #FFF3A3A6;">both are consumed</mark>
- <mark style="background: #FFF3A3A6;">If the focus is NT,</mark> <mark style="background: #FFF3A3A6;">consult parse table</mark>,<mark style="background: #FFF3A3A6;"> determine the correct production</mark> <mark style="background: #FFF3A3A6;">according to</mark> the <mark style="background: #FFF3A3A6;">Start set</mark>. 



A grammar is considered **backtrack-free** if, for any nonterminal A with multiple alternative right-hand sides, the FIRST+ sets of those alternatives are pairwise disjoint (all disjoint). This is known as an **LL(1) grammar.**

For every terminal $w \in FIRST^+$ $(A \to \beta)$, the table entry $Table[A, w]$ is set to the production $A \to \beta$. 
