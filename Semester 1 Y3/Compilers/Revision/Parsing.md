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
- Given productions of the form $A → αβ₁ | αβ₂ | αβ₃$ where $α$ is the **common prefix** and $β_i$ are the distinct suffixes.
- Left factor them into:
	- $A → a  