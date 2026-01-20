## Handling whitespace
- <mark style="background: #FFF3A3A6;">Define whitespace</mark> in lexical grammar <mark style="background: #FFF3A3A6;">like any other token</mark>
- What we do depends on lang
	- If <mark style="background: #FFF3A3A6;">carries no meaning</mark>, then <mark style="background: #FFF3A3A6;">don't emit token</mark>; <mark style="background: #FFF3A3A6;">otherwise emit</mark> token like everything else with whitespace lexeme and syntactic category/kind
## Scanners vs Automata
- <mark style="background: #FFF3A3A6;">Typical DFA</mark>, tries to <mark style="background: #FFF3A3A6;">recognise</mark> the <mark style="background: #FFF3A3A6;">entire input</mark>
	- A <mark style="background: #FFF3A3A6;">scanner</mark> needs to <mark style="background: #FFF3A3A6;">repeatedly recognise</mark><mark style="background: #FFF3A3A6;"> portions of the input</mark>,<mark style="background: #FFF3A3A6;"> identifying the longest prefix </mark>from the remaining **input that <mark style="background: #FFF3A3A6;">forms a valid token. </mark>**
- **DFA has to report 'reject' or 'accept'**
	- A theoretical DFA only answers "accept or reject"; a <mark style="background: #FFF3A3A6;">scanner's "DFA" must say</mark> which token was accepted/<mark style="background: #FFF3A3A6;">which state was accepted in.</mark>
- **If a DFA reaches an error state it immediately rejects** 
	- <mark style="background: #FFF3A3A6;">Scanner may be able to ‘rollback</mark>’ to find a shorter prefix it could accept


### Hopcroft's Algorithm
- Performs $DFA$<mark style="background: #FFF3A3A6;"> minimisation</mark>, transforming given DFA into equivalent DFA that has minimum number of states.<mark style="background: #FFF3A3A6;"> Two DFAs</mark> are <mark style="background: #FFF3A3A6;">equivalent</mark> in that they <mark style="background: #FFF3A3A6;">recognise</mark> the <mark style="background: #FFF3A3A6;">same regular language</mark>.
- Hopcroft's algorithm, concerned, <mark style="background: #FFF3A3A6;">merging nondistinguishable</mark> states of $DFA$ based on <mark style="background: #FFF3A3A6;">partition refinement</mark>; divides into separate steps
- Each partition must follow 2 rules:
	- For<mark style="background: #FFF3A3A6;"> each state in a partition</mark>, <mark style="background: #FFF3A3A6;">every transition</mark> <mark style="background: #FFF3A3A6;">must lead</mark> to <mark style="background: #FFF3A3A6;">another state</mark> in the <mark style="background: #FFF3A3A6;">same partition</mark>
	- The<mark style="background: #FFF3A3A6;"> states in a partition</mark> must<mark style="background: #FFF3A3A6;"> either</mark> be <mark style="background: #FFF3A3A6;">all accepting</mark> or <mark style="background: #FFF3A3A6;">all nonaccepting</mark>
- Partitions then represent one state in the minimised DFA, equivalence classes, same behaviour for every input sequence.

#### Practical example 
1. Initial partition:
	Set of final states and non-final states placed into two separate partitions.
	![](Pasted%20image%2020260117224915.png)
2. For each symbol in the alphabet, determine all states that can reach a state in $p$(the image), for each set $q$ in the partition that has one of those states, compute $q1 = q \cap image$(actual states that do the transition), $q2 = q - q1$(remove the states). If q2 nonempty, remove $q$ from the partition and add $q1$ and $q2$
3. Keep going until no sets need to be split. 

Intuitively, two states are equivalent if every possible future input leads to same accept/reject outcome.

### Limitations
- In a scanner, <mark style="background: #FFF3A3A6;">final state</mark> <mark style="background: #FFF3A3A6;">correspond</mark> to<mark style="background: #FFF3A3A6;"> token classes,</mark> if we <mark style="background: #FFF3A3A6;">place them in the same partition</mark> at any point, then we<mark style="background: #FFF3A3A6;"> effectively lose the ability to distinguish</mark>. Can <mark style="background: #FFF3A3A6;">modify the algorithm</mark> and<mark style="background: #FFF3A3A6;"> place each final state</mark> in a <mark style="background: #FFF3A3A6;">separate set</mark> to begin with. 
### Implementing scanners
- Differing common implementation strategies, <mark style="background: #FFF3A3A6;">all operate in same manner</mark> by simulating the DFA. 
- <mark style="background: #FFF3A3A6;">Process stops</mark> when $DFA$ <mark style="background: #FFF3A3A6;">recognises a word,</mark> which <mark style="background: #FFF3A3A6;">occurs when </mark><mark style="background: #FFF3A3A6;">current state</mark> $s$ has <mark style="background: #FFF3A3A6;">no outbound transition</mark> on current input character, <mark style="background: #FFF3A3A6;">if </mark>$s$ is <mark style="background: #FFF3A3A6;">accepting,</mark> scanner <mark style="background: #FFF3A3A6;">recognises</mark>, returns token. 
- If $s$ non-accepting, determine if passed through to accepting state on way to $s$, if it did,<mark style="background: #FFF3A3A6;"> roll back internal state + input stream</mark>, report success.
- If not, report failure.

#### Table-Driven Scanners
- <mark style="background: #FFF3A3A6;">Uses lookup table</mark> <mark style="background: #FFF3A3A6;">determine which action to take</mark>, s<mark style="background: #FFF3A3A6;">imulating transition table</mark>/function of DFA. 
- Further tables are used to:
	- <mark style="background: #FFF3A3A6;">Group characters</mark> into **character classes**, make the table smaller
	- <mark style="background: #FFF3A3A6;">Indicate which final states</mark> <mark style="background: #FFF3A3A6;">match which token classes.</mark> 
- Each character lookup is done by indexing into a 2d array `next_state = transition_table[current_state][input_char]` (or char_category as j index.)

##### Character classes
- Often, many diff characters involve same transition
- Trad approach, one column for each char
- Instead, group characters into classes e.g., digit, lowercase letter.
- Look up the current characters class in character class table before transitioning. 

##### Token class table
Often have token type/lexical category table which maps each final DFA state to the lexical category it represents. 

##### Transition table example
![[Pasted image 20251010191654.png]]


##### Rollback
- <mark style="background: #FFF3A3A6;">Must consider rollback loop</mark>
- Consider "123abc"
	- Not valid number or identifier, but contains both
- <mark style="background: #FFF3A3A6;">Scanners need to be able to recover</mark>; <mark style="background: #FFF3A3A6;">as ingesting input,</mark> we <mark style="background: #FFF3A3A6;">maintain a stack of states</mark>; we <mark style="background: #FFF3A3A6;">place</mark> the <mark style="background: #FFF3A3A6;">states seen</mark> <mark style="background: #FFF3A3A6;">onto a stack</mark>, if end of input word is reached, and <mark style="background: #FFF3A3A6;">state is not accepting</mark>, pop() to<mark style="background: #FFF3A3A6;"> 'rollback' until reach accepting state whilst truncating lexeme</mark>, then return valid token according to most recent previous accepting state. 
 ![](Pasted%20image%2020260118000854.png)

##### Table-driven performance
- Two main bottlenecks
	1. <mark style="background: #FFF3A3A6;">Acquiring characters from input</mark>
	2. <mark style="background: #FFF3A3A6;">Lookup values in table</mark>
- <mark style="background: #FFF3A3A6;">Direct coded scanners</mark><mark style="background: #FFF3A3A6;"> reduce overhead of table lookups</mark>
	- Use <mark style="background: #FFF3A3A6;">goto statements</mark> or <mark style="background: #FFF3A3A6;">recursive methods </mark>to directly represent transitions.
- Also have the rollback issue; certain DFAs and rollback strategies can result in quadratic time spent reading the input stream. 

#### Direct coding
- <mark style="background: #FFF3A3A6;">To improve the performance of a table-driven scanner,</mark> must <mark style="background: #FFF3A3A6;">reduce the cost of one or both of its basic actions: </mark>
	1. <mark style="background: #FFF3A3A6;">Read a character</mark>
	2. <mark style="background: #FFF3A3A6;">Compute </mark>next DFA <mark style="background: #FFF3A3A6;">transition</mark>
- Replace transition table, with implicit representation, to <mark style="background: #FFF3A3A6;">reduce overhead per character</mark>
	- Warranted, each char, table-driven scanner performs address computations, and 2 load operations.
- To achieve, embed DFA logic into program control stucture
- Each DFA state becomes block of code, transitions become goto, switch, and nested if/else. 

![](Pasted%20image%2020260118001526.png)
![](Pasted%20image%2020260118001521.png)
But would have same rollback mechanism, err would encode this, and each label would start with pushing that state onto the stack. 