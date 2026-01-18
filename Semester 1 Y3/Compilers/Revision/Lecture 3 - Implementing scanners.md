## Handling whitespace
- Define whitespace in lexical grammar like any other token
- What we do depends on lang
	- If carries no meaning, then don't emit token; otherwise emit token like everything else with whitespace lexeme and syntactic category/kind
## Scanners vs Automata
- Typical DFA, tries to recognise the entire input
	- A scanner needs to repeatedly recognise portions of the input, identifying the longest prefix from the remaining input that forms a valid token. 
- DFA has to report 'reject' or 'accept'
	- A theoretical DFA only answers "accept or reject"; a scanner's "DFA" must say which token was accepted/which state was accepted in.
- If a DFA reaches an error state it immediately rejects 
	- Scanner may be able to ‘rollback’ to find a shorter prefix it could accept


### Hopcroft's Algorithm
- Performs $DFA$ minimisation, transforming given DFA into equivalent DFA that has minimum number of states. Two DFAs are equivalent in that they recognise the same regular language.
- Hopcroft's algorithm, concerned, merging nondistinguishable states of $DFA$ based on partition refinement; divides into separate steps
- Each partition must follow 2 rules:
	- For each state in a partition, every transition must lead to another state in the same partition
	- The states in a partition must either be all accepting or all nonaccepting
- Partitions then represent one state in the minimised DFA, equivalence classes, same behaviour for every input sequence.

#### Practical example 
1. Initial partition:
	Set of final states and non-final states placed into two separate partitions.
	![](Pasted%20image%2020260117224915.png)
2. For each symbol in the alphabet, determine all states that can reach a state in $p$(the image), for each set $q$ in the partition that has one of those states, compute $q1 = q \cap image$(actual states that do the transition), $q2 = q - q1$(remove the states). If q2 nonempty, remove $q$ from the partition and add $q1$ and $q2$
3. Keep going until no sets need to be split. 

Intuitively, two states are equivalent if every possible future input leads to same accept/reject outcome.

### Limitations
- In a scanner, final state correspond to token classes, if we place them in the same partition at any point, then we effectively lose the ability to distinguish. Can modify the algorithm and place each final state in a separate set to begin with. 
### Implementing scanners
- Differing common implementation strategies, all operate in same manner by simulating the DFA. 
- Process stops when $DFA$ recognises a word, which occurs when current state $s$ has no outbound transition on current input character, if $s$ is accepting, scanner recognises, returns token. 
- If $s$ non-accepting, determine if passed through to accepting state on way to $s$, if it did, roll back internal state + input stream, report success.
- If not, report failure.

#### Table-Driven Scanners
- Uses lookup table determine which action to take, simulating transition table/function of DFA. 
- Further tables are used to:
	- Group characters into **character classes**, make the table smaller
	- Indicate which final states match which token classes. 
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
- Must consider rollback loop
- Consider "123abc"
	- Not valid number or identifier, but contains both
- Scanners need to be able to recover; as ingesting input, we maintain a stack of states; we place the states seen onto a stack, if end of input word is reached, and state is not accepting, pop() to 'rollback' until reach accepting state whilst truncating lexeme, then return valid token according to most recent previous accepting state. 
 ![](Pasted%20image%2020260118000854.png)

##### Table-driven performance
- Two main bottlenecks
	1. Acquiring characters from input
	2. Lookup values in table
- Direct coded scanners reduce overhead of table lookups
	- Use goto statements or recursive methods to directly represent transitions.
- Also have the rollback issue; certain DFAs and rollback strategies can result in quadratic time spent reading the input stream. 

#### Direct coding
- To improve the performance of a table-driven scanner, must reduce the cost of one or both of its basic actions: 
	1. Read a character
	2. Compute next DFA transition
- Replace transition table, with implicit representation, to reduce overhead per character
	- Warranted, each char, table-driven scanner performs address computations, and 2 load operations.
- To achieve, embed DFA logic into program control stucture
- Each DFA state becomes block of code, transitions become goto, switch, and nested if/else. 

![](Pasted%20image%2020260118001526.png)
![](Pasted%20image%2020260118001521.png)
But would have same rollback mechanism, err would encode this, and each label would start with pushing that state onto the stack. 