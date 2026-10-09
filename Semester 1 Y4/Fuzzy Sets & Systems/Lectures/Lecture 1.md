## Set Operations recap
- **Union**
	- $A \cup B$ , set of all elements of $A$ and $B$ 
- **Intersection**
	- $A \cap B$, set of all elements present in both $A$ and $B$
- **Difference**
	- $A \textbackslash B$ i.e., the set containing all elements belong to $A$ but not $B$.
### Boolean Logic recap
- Two truth values: $TRUE$, $FALSE$ 
- Logical connectives: conjunction, disjunction, negation, implication, equivalence
	![](Pasted%20image%2020261009013938.png)
- Rules:
	**Commutativity:**
		$p \vee q \iff q \vee p$  
		$p \wedge q \iff q \wedge p$  
	**Associativity**
		$p \vee (q \vee r) \iff (p \vee q) \vee r$ 
		$p \wedge (q \wedge r) \iff (p \wedge q) \wedge r$ 
	**Distributivity**
		$p \vee (q \wedge r) \iff (p \wedge q) \vee (p \wedge r)$ 
		$p \wedge (q \vee r) \iff (p \vee q) \wedge (p \vee r)$ 
	**First De Morgan Law**
		$\neg(p \wedge q) \iff \neg p \vee \neg q$
	**Second De Morgan Law**
		$\neg (p \vee q) \iff \neg p \wedge \neg q$ 


## Characteristic (membership) Functions
- The membership of an element in a set can be described by a characteristic function, often referred to as a membership function. 
- For a given set $A$ this function assigns a value $\mu_{A}(x)$ to every element $x$ in the universal set $X$ such that:
	- $\mu_{A}(x) = 1 \iff x \in A$ 
	- $\mu_A(x) = 0 \iff x \notin A$ 
- In other words, the characteristic maps elements of the universal set to the set $\{0, 1\}$ and subsequently describes set membership:
	- $\mu_{A}(x):X \to \{0,1\}$ 
- A binary membership/characteristic function however can be limiting, describing heaps of sand for example, how much sand does one need to become a heap? 
	![](Pasted%20image%2020261009015222.png)
	If the value of say, 300,000 is needed to become a 'heap' of sand, and this is where $\mu_A(x)$ is $1$ then set membership is not a good description. 

### Alternative characteristic function
- We need a mapping beyond that which is merely binary
- We can map elements from the universal set $X$ to a real value in $[0, 1]$ i.e., for the set ***Heap*** we have:
	- $\mu_{Heap}(x): X \to [0, 1]$ 
- So we can precisely define the membership in the set Heap for say, 2 grains of a sand, and 200,000 grains of sand. 

### Parametric Characteristic Functions
- **Triangular Membership Function**
	- Defined by three parameters $a \leq b \leq c$: the two 'feet' $a$ and $c$ describe where the membership begins and ends(in relation to the universe of discourse), and the peak $b$.
	$$0, x \leq a, x \geq c$$
$$  
\mu_A(x) = \max\!\left(\min\!\left(\frac{x-a}{b-a},\ \frac{c-x}{c-b}\right),\ 0\right)  
$$
	As $x$ grows, so does the first fraction, until reach the peak specified at $b$ (where $x \geq b$), then take the second fraction. 
		![](Pasted%20image%2020261009021153.png)

- **Trapezoidal Membership Function** 
	- Defined by four parameters: $a \leq b \leq c \leq d$, and only differs to the above by providing a flat top with the further parameter, which suits concepts that describe a range e.g., a comfortable room temperature is between 19 and 23 degrees.
		![](Pasted%20image%2020261009021202.png)
$$\mu_A(x) = \max\!\Big(\min\!\Big(\underbrace{\tfrac{x-a}{b-a}}_{\text{row 1}},\ \underbrace{1}_{\text{row 2}},\ \underbrace{\tfrac{d-x}{d-c}}_{\text{row 3}}\Big),\ \underbrace{0}_{\text{"otherwise"}}\Big)$$
Row 2 i.e., the flat top of the trapezoid is where: $x \geq b, x \leq c$ 

## Fuzzy set
- Given a universe of discourse $X$, a **Fuzzy Set** $A$ is defined by its **membership function**
- That is, the elements of the universe of discourse $X$ are 'characterised' by a membership function to form set Fuzzy set $A$ usually denoted as a set of ordered pairs:$$A = \{(x, \mu_A(x)) \ \vert\  x \in X\}$$
- Where $\mu_A(x)$ is a value yielded by the membership function that describes the degree of membership of $x$ in $A$:$$\mu_{A}:X \to [0,1]$$
- A 'classical' (**crisp**) set is the special case where $\mu_{A}(x)$ only takes values in $\{0,1\}$; it's classification of membership is binary. 
- Every element in the universe of discourse $X$ is characterised by the membership function, even if that characterisation means that the element is assigned a degree of membership=0. 
