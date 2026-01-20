### Scanner
- Process,<mark style="background: #FFF3A3A6;"> turning input stream</mark> of <mark style="background: #FFF3A3A6;">charcters</mark>, into meaningful <mark style="background: #FFF3A3A6;">lexical tokens</mark>, via <mark style="background: #FFF3A3A6;">aggregation of characters into lexemes</mark>, which are<mark style="background: #FFF3A3A6;"> assigned a syntactic category</mark>/token kind.
- The <mark style="background: #FFF3A3A6;">lexical grammar</mark> defines what counts as a token and what syntactic category belongs to that particular kind of token. 
- <mark style="background: #FFF3A3A6;">Scanning may parse to add semantic value along with, or instead of lexeme e.g.,{ "123", "literal"} -> {123, "literal"}</mark>
- Scanning yields all these tokens.


### Recognisers and regular expressions
**Regex**, convenient way of<mark style="background: #FFF3A3A6;"> describing</mark> a class of words, that is,<mark style="background: #FFF3A3A6;"> regular languages</mark>. These are parallel with **finite automation**, but are more intuitive for humans to use. 

**Regex** rules(significantly less formal than usual):
- a = single "a"
- "ab" = a followed by b
- "a|b" = a or b
- "a*" = arbitrary number of a's
- "a+" = at least one a

**Regex examples**
- `var`
	- The string "var"
- `0|1|2|3|4|5|6|7|8|9`
	- Any single digit
	- Commonly written as `[0-9]`
- `[a-z][a-zA-Z]*`
- `(0x|0X)[0-9a-fA-F]+`
<mark style="background: #FFF3A3A6;">FREE TO USE RANGES IN EXAM</mark>

**Lexical grammars** <mark style="background: #FFF3A3A6;">correspond</mark> to a list of <mark style="background: #FFF3A3A6;">regular expressions</mark>, and the <mark style="background: #FFF3A3A6;">token classes</mark> they correspond to. When a scanner recognises a string matching a regular expression, <mark style="background: #FFF3A3A6;">returns</mark> the<mark style="background: #FFF3A3A6;"> token</mark> consisting of the <mark style="background: #FFF3A3A6;">matching lexeme</mark> and the <mark style="background: #FFF3A3A6;">token</mark> name/<mark style="background: #FFF3A3A6;">class</mark>.

The goal of working with finite automata; automation derivation of executable scanners from collection of $REs$. Develop constructions, transform $RE \to FA$.  
	THE GENERAL METHOD:
	- **Thompson's construction**: $RE \to NFA$ 
	- **Subset construction**: $NFA \to DFA$ 
	- **Hopcroft's algorithm**: Minimises a $DFA$
- Kleene's construction may be used to derive an $RE$  from a $DFA$ 
To yield a scanner, we simulate the DFAs that are produced from this pipeline; via alternation/union we combine them to simulate them as one entity. 

### Transition diagrams
- Visually represent automaton, states = circles, start state has arrow coming into it, accepting states have double rings, or state name is underlined. 
### Thompson's Construction
Mechanical method, $RE \to NFA$

The following rules are depicted:
1. <mark style="background: #FFF3A3A6;">Empty word</mark>
	![[Pasted image 20251007234417.png]]
2. <mark style="background: #FFF3A3A6;">Symbo</mark>l $a$ 
	![[Pasted image 20251007234437.png]]
		E.g., $\alpha \xrightarrow{v} \beta \xrightarrow{a} \gamma \xrightarrow{r} \omega$  for $var$, just simply chained (though this is technically concatenation)
3. $Union$
	![[Pasted image 20251007234514.png]]
	Cba with the latex
	Have start state, silent transition, starts of both NFAs derived for the REs, final states of both REs merge at the final state of the entire NFA (in ians slides, dont merge at end)
		![](Pasted%20image%2020260117214931.png)
4. $Concatenation$
	$α \xrightarrow{a} β \xrightarrow{\epsilon} γ \xrightarrow{b} ω$
	Start state of $N(s)$ is initial state of whole $NFA$, silent transition to start state of $N(t)$, whose final state is the final state of the whole $NFA$
5. $Kleenestar$ 
	![](Pasted%20image%2020260117214958.png)
	Treat as singular symbol/word, then add dummy state, and 2 empty transitions
### Subset Construction
- Not gonna be as mechanical as LAC
- Differs slightly; we have $\epsilon$ transitions in our $NFAs.$
- Must take $\epsilon$ closures when computing subset construction over an $NFA$ 
1. Start with set of $NFA$ states
2. Follow input symbol from initial state
3. Take e closure of the result
4. That closure is the next DFA state
5. Repeat until no new sets appear in LHS
Now have table




![](Pasted%20image%2020260117220614.png)






# Questions
- Give NFA for a(b|c)*de
- etc, answers in book. 