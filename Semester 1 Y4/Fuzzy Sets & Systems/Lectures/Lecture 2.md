## 1965 Paper Zazeh's Fuzzy sets
- Zadeh noted that many categories used in everyday reasoning have no sharp, binary boundary such as 'tall men' or 'numbers greater than 1'. 
- His own exxample was the class of animals, as this includes cats, dogs etc, clearly excludes rocks, and things like starfish or bacteria are ambiguous.
- Classical sets force a binary answer, which misrepresents our reality.
- A fuzzy set $A$ in $X$ is characterised by a membership function assigning each $x$ a grade of membership in $[0,1]$. 
	- Zadeh wrote this as  $f_A(x)$; $\mu_A(x)$ which would become the later standard.
- Set operations were redefined in terms of membership values, aka applied **pointwise**(element by element):
	- **Set Containment**
		$A \subseteq B$ if $\mu_A(x) \leq \mu_{B}(x), \ \forall x \in A,B$
		An element cannot belong to $A$ more than it belongs to $B$.
	- **Set Union**
		$\max(\mu_A(x), \mu_B(x))$
	- **Set Intersection**
		$\min(\mu_A(x), \mu_B(x))$
	- **Set Complement**
		$1 - \mu_A(x)$
- So to form a new fuzzy set $C$ from the results of one of these operations(besides set containment, which is a logical operation and does not produce a new set) we would apply to each element's membership value $\mu_{X}(x)$ where $X$ is an arbitrary set pointwise to yield a new set of membership values for corresponding elements in the domain of discourse, effectively defining a new fuzzy set with no direct membership function. 

### Probability vs Fuzzy sets
- Probability describes uncertainty about whether something is the case; the underlying fact is crisp but your knowledge of the answer is incomplete.
- Fuzzy membership describes *vagueness* i.e., how much a property holds. 


### $\alpha$-cuts
- An important concept which establishes a relationship between a **crisp set** and a **fuzzy set** is the concept of an $\alpha$**-cut**.
- An $\alpha$-cut of a fuzzy set $A$ is a crisp set $A$ that contains all the elements of $A$ with membership greater than or equal to the specified value of $\alpha$. 
	$$C_{\alpha} = \{x \in X \ \vert \ \mu_A(x) \geq \alpha\}$$ 
- The **strong** $\alpha$-cut is defined as:
	This naturally only differs from the above where $\mu_A(x) = \alpha$$$A_\alpha^+ = \{\, x \in X \mid \mu_A(x) > \alpha \,\}$$
- Normalises elements of $A$ to a yes/no in $A_{\alpha}$ according to $\mu_A(x)$ and $\alpha$.
	- Written as a membership function, an $\alpha$ cut only ever outputs $0$ or $1$.
- ![](Pasted%20image%2020261009033237.png)
- All elements between 1.5 and 4.2 in $X$ are in $A$, whereas prior to taking the $\alpha$-cut they had differing degrees of membership.  

### Support
- The **support** is the strong $\alpha$-cut of $A$ for $\alpha$ = 0
- That is, every element with any membership at all:$$supp(A) = (x \in X \ \vert \ \mu_{A}(x) > 0)$$
- We naturally need the strong cut because every $x \in X$ has $\mu_A(x) \geq 0$ and would effectively yield $X$ as the support of $A$. 


### Normality
- The height of a fuzzy set is the largest membership grade attained by any element that is a member of that set.
- A fuzzy set is called **normal** if at least one of its elements attains the **maximum possible grade of membership** i.e., 1 if grades are between $[0,1]$ 
- Fuzzy sets can be **normalised** i.e., converted(usually scaled) so they are **normal**. 
- A fuzzy set does not have to be normal, but some special kinds of fuzzy sets do require normality.
![](Pasted%20image%2020261009034127.png)
### Convexity
- A fuzzy set $A$ is **convex** if and only if:$$\mu_(A)(\lambda r + (1 - \lambda) s) \geq min(\mu_A(r), mu_A(s))$$$$\forall r, s \in X \quad\text{and} \quad \forall \lambda \in [0,1]$$
- If we take two arbitrary elements in the universe of discourse, and ask whether the degree of membership obtained by varying $\lambda$ between these 2 points **does not** exceed the degree of membership of these two arbitrary elements $r$ and $s$ then we can say the set is not convex because it has a dip(degree of membership where 2 elements next to it have higher degrees of membership).
	![](Pasted%20image%2020261009040046.png)
- A fuzzy set is **convex** exactly when every single one of its $\alpha$-cuts is a convex set. 


### Convexity of sets vs convexity of functions
![](Pasted%20image%2020261009040827.png)
![](Pasted%20image%2020261009040833.png)




## Basic Operations
### Complement
- The complement $\bar{A}$ of a fuzzy set $A$ is given by $$\mu_{\bar{A}}(x) = 1 - \mu_{A}(x)$$
- Logical NOT, merely inverses the membership degree of element $x \in X$ 
- If element 25 belongs to $A$ with degree 0.7, then it belongs to the complement of $A$ with degree 0.3.
	![](Pasted%20image%2020261009041210.png)

### Intersection
- The intersection $A \cap B$ of two fuzzy sets is given by: $$\mu_{A \cap B}(x) = \star(\mu_{A}(x), \mu_{B}(x))$$
	Where $\star$ is a **conorm**, that is, the name for the family of functions allowed to act as $AND$. Min is Zadeh's original choice. Product is another option, but will yield differing degrees of membership for given $x \in X$ that is also a member of $A$ or $B$ or both. 
- Min makes sense, if not in one set, then $x$ will be zero and not a member of A AND B.

### Union
- The **union** of two fuzzy sets is commonly given by: $$\mu_{A \cup B}(x) = \oplus\left(\mu_A(x), \mu_B(x)\right), \text{ where } \oplus \text{ is a t-conorm}$$
- That is,  the family of functions permitted to act as $OR$.


![](Pasted%20image%2020261009042026.png)
![](Pasted%20image%2020261009042034.png)

## Parameterised Operations

### Complement
- A complement $c$ is a function which converts a fuzzy set $A$ to another fuzzy set $\bar{A}$:$$c:[0,1] \to [0,1], \forall x \in X: \mu_{\bar{A}}(x) = c(\mu_{\bar{A}}(x))$$
- To behave sensibly as a logical NOT, the function $c$ must satisfy the following axioms:
	- **Axiom c1** - **Boundary Conditions**
		 $c(0) = 1$ and $c(1) = 0$ (behaves like crisp sets - inverting the membership degree)
	- **Axiom c2** - **Monotonic non-increasing**
		$\forall a, b \in [0,1]:$ if $a \leq b$ then $c(a) \geq c(b)$ where $a = \mu_{A}(x_{1})$ and $b = \mu_{A}(x_{2})$
		That is to say, the complement of the membership degrees of some elements $x_1, x_2 \in X$ should have the degree to which they are in the fuzzy set inverted with respect to one another
		The more something belongs to $A$, then the less it can belong to 'not $A$'. It would make no sense for $x_2$ to be more in $A$ than $x_1$ and also be more in $\bar{A}$ than $x_1$ too, vice-versa.
		Wherever you are on the curve of $\bar{A}$ as characterised by $c$, everything to the right is the same height or lower.
	- **Axiom c3** - **Continuity**
		$c$ should be a continuous function. A tiny change in membership should only cause a tiny change in the complement. 
	- **Axiom c4** - **Involution**
		- $c$ should be involute, that is $c(c(a)) = a, \forall a \in [0,1]$ 
- All fuzzy complements satisfy the required axioms $c1$ and $c2$.
- Continuous complements are complements that further satisfy $c3$.
- All functions satisfying axioms $c4$ and $c2$, form a further nested sub-class of complements as they are automatically continuous(can't have any gaps because of involute $\forall a \in [0,1]$ and cannot 'climb back up' because of $c2$)

![](Pasted%20image%2020261010005112.png)
Graph 2 and 3 are the same, just different terms. These functions satisfy the above axioms and as thus can behave as logical not (as demonstrated by being drawn)

### Intersection
- Intersections of Fuzzy Sets are repesented by an established class of functions called **triangular norms** or '**t-norms**'
- A **t-norm** $\star$ is a function which takes two arguments in $[0,1]$ and returns a value in $[0,1]$:$$\star : [0,1] * [0,1] \to [0,1]$$
- Thus, we can write:$$\mu_{A}(x) \cap \mu_{B}(x) = \star(\mu_{A}(x), \mu_{B}(x))$$
#### Axiomatic Basis for t-norm(and thus, intersection)
- Consider $a,b,c \in [0,1]$. A t-norm must satisfy the following axioms:
	- Axiom $t1$: **Boundary Conditions**
		$\star(a, 1) = a$
		Performing AND with something that has maximal membership changes nothing; the result is just your membership e.g., $p \wedge TRUE = p$ 
	- Axiom $t2$: **Monotonicity**
		$b \leq c \to \star(a,b) \leq \star(a, c)$ 
		if $b \leq c$ then applying the $t-norm$ with respect to some other fixed $a$ for both $b$ and $c$ will result in value(s) that maintains that relationship. This is true of classical AND.
	- Axiom $t3$: **Commutativity**
		$\star(a, b) = \star(b,a)$ 
		Order is irrelevant.
	- Axiom $t4$: **Associativity**
		$\star(a, \star(b,c)) = \star(\star(a, b), c)$ 
		Grouping does not matter permitted they are in the same order.

#### Optional Intersection axioms
- **Continuity**
	A tiny change in either membership value should only cause a tiny change in the result. 
- **Idempotence**
	$\star(a, a) = a$ 

#### Common t-norms
- **Minimum**$$a \star b = min(a, b)$$
	This is the *only* t-norm satisfying $t1$ to $t6$ 
- **Product**$$a \star b = ab$$ 
- **Bounded difference**$$a \star b = max(0, a+b-1)$$
- **Drastic product** $$a\star b = \begin{cases} a & \text{if } b = 1 \\ b & \text{if } a = 1 \\ 0 & \text{otherwise} \end{cases}$$
	Harshest AND possible as membership is only yielded where the other argument to the t-norm is 1 i.e., has maximal membership.


### Union
- Unions of fuzzy sets are represented by an established class of functions called **triangular co-norms** or **t-conorms**
- These functions depict classical OR/set union in Fuzzy logic/sets
- A **t-conorm** is a function which takes **two arguments** in $[0, 1]$ and returns a value in $[0,1]$:$$\oplus: [0, 1] * [0,1] \to [0,1]$$
- Thus, we can write:$$\mu_A(x) \cup \mu_B(x) = \oplus(\mu_A(x), \mu_B(x))$$
#### Axiomatic basis for t-conorm(and thus, union)
- Consider $a,b,c \in [0,1]$ 
- A **t-conorm** must satisfy the following axioms:
	- Axiom $u1$: **Boundary Conditions**
		$\oplus(a, 0) = a$ 
		Entirely dependent on the incoming value where one input is 0/not a member of the fuzzy set. 
	- Axiom $u2$: **Monotonicity**
		$b \leq c \to \oplus(a, b) \leq \oplus(a, c)$ 
		Maintains size relationship when applying the function with an independent variable; the function should not warp the original relationship between the membership degrees.
	- Axiom $u3$: **Commutativity**
		$\oplus(a, b) = \oplus(b,a)$ 
		Order of arguments does not matter.
	- Axiom $u4$: **Associativity**
		$\oplus(a, \oplus(b,c)) = \oplus(\oplus(a, b), c)$ 
		Grouping of application/arguments does not affect the final output. 

Continuity and idempotence also remain as optional $t-conorm$ axioms, much like $t-norms$ 


#### Common t-conorms
- **Maximum**
	$a \oplus b = max(a, b)$ 
	Only function to satisfy all 6 axioms. 
- **Probabilistic Sum**
	$a \oplus b = a + b - ab$ 
- **Bounded sum**
	$a \oplus b = min(1, a+b)$ 
- **Drastic sum**
	$a \oplus b = a \ if \ b = 0;\  b \ if \ a = 0; \ otherwise \ 1$ 
	Models OR naturally, OR should just be membership in a fuzzy set and is thus the most generous OR possible whilst still obeying the axioms.  



## Linguistic Variables
- A **linguistic variable** is a collection of fuzzy sets representing the linguistic terms of a concept.
- An ordinary variable takes **numbers** as values e.g., height = 1.75m.
- A **linguistic variable** takes **words** as values: height = 'tall' and each word/term describes a fuzzy set to collectively describe the overarching concept.
- This allows us to reason with words the way people do, whilst being able to calculate. E.g., IF temperature is high THEN fan speed is fast. 
	- E.g., temperature and fan speed are linguistic variables, with respective fuzzy sets denoting terms that describe those variables e.g., high, low, fast, slow.
- We can define 'height' in terms of 3 fuzzy sets:
	- **Short**
	- **Medium**
	- **Tall**
- For discrete measurements, we can define the degree of membership in each of these sets:
	![](Pasted%20image%2020261010023457.png)
### Formally
- A linguistic variable is characterised by a quintuple:$$\chi, T(\chi), U, G, M$$
- $\chi$: The name of the linguistic variable
- $T(\chi)$: A set of terms that describe the linguistic variable
- $U$: The universe of discourse; the range of underlying values for the linguistic variable $\chi$ 
- $G$: A syntactic rule, which often takes the form of a grammar, for generating the terms e.g., very + a term, not+a term
	- Decides which terms are valid.
- $M$: A semantic rule which associates each linguistic term $X$ in $T(\chi)$ its meaning, $M(X)$, which denotes a fuzzy set.
	- Describes what terms mean concretely. 

#### Example
- Consider a linguistic variable $\chi$ named $Age$, $\chi = Age$ 
- It is defined over a universe of discourse $U = [0, 100]$ 
- The term set $T$ associated with age may be:
	𝑇 = 𝑦𝑜𝑢𝑛𝑔 + 𝑣𝑒𝑟𝑦 𝑦𝑜𝑢𝑛𝑔 + 𝑛𝑜𝑡 𝑦𝑜𝑢𝑛𝑔 + 𝑚𝑖𝑑𝑑𝑙𝑒 − 𝑎𝑔𝑒𝑑 + 𝑛𝑜𝑡 𝑚𝑖𝑑𝑑𝑙𝑒 − 𝑎𝑔𝑒𝑑 + 𝑜𝑙𝑑 + 𝑣𝑒𝑟𝑦 𝑜𝑙𝑑 + 𝑛𝑜𝑡 𝑜𝑙𝑑 + 𝑦𝑜𝑢𝑛𝑔 𝑜𝑟 𝑚𝑖𝑑𝑑𝑙𝑒 − 𝑎𝑔𝑒𝑑 + 𝑛𝑜𝑡 𝑣𝑒𝑟𝑦 𝑜𝑙𝑑 + . . .
- Some terms are **atomic**
	- Their semantics have been described directly by the designer; a membership function has been designed for them
- Some terms are **composite**
	- Their semantics are described via instantiating a new fuzzy set from the constituent atomic parts of the composite term $X$
	- E.g., $very$ $old$ is not directly defined, we define old and then mutate $M(old)$ according to $very$ 

### Words and Fuzzy Sets
![](Pasted%20image%2020261010025113.png)

### Hedge
- A hedge is a qualifying word added to a term to indicate a minor modification of the usual meaning of the term
- In english common hedges are words such as: extremely, very, rather, quite, not, somewhat, not as, more or less etc...

### Concentration
- Squaring a 'membership function' (more accurately, squaring the outputs of a membership function for some $M(X)$ with respect to $U$) makes it more 'concentrated'
- Strong members keep most of their membership, whilst weak members lose most of theirs, proportional to their degree of membership this is.
- The shape of the fuzzy set becomes more focused on the elements that belong to $M(X)$ more strongly
	![](Pasted%20image%2020261010025938.png)
- $very$: $\mu_{very_{A}}(x) = (\mu_{{A}}(x))^2$ 

### Dilation
- Square-rooting a membership function's outputs makes it 'less concentrated' i.e., **diluted**
- Multiplies values between $0$ and $1$ (who retain their values) proportional to their degree of membership in $M(X)$, doing the opposite of the above.
- The shape of the fuzzy set becomes lenient towards members with lower degrees of membership, the shape is relaxed.
	![](Pasted%20image%2020261010030429.png)
- $slightly = \mu_{slightly_{A}}(x) = \sqrt{\mu_{{A}}(x)  }$ 

### Meaning of terms
- Often, the meaning of terms such as $M(young)$ are defined by  functions
	- This is not necessarily the case
	- Could be defined by enumeration/look-up
	- Depends entirely on the values in $U$ 
- When a mapping from elements of $U$ to membership grades is defined, it is called a membership function
- Thus, the set $M$ can be viewed as the collection of membership functions that associate linguistic terms permitted by the grammar $G$with their meanings. 
- The relationship between the terms, and the functions that describe them fall apart very quickly if not designed well e.g., why square to concentrate, why not cube? 
	- Fuzzy systems are often promoted as explainable systems because their rules read like sentences.
	- A rule is only a genuine explanation if the words mean what a human reader thinks they mean, which requires careful selection of terms, and careful design of membership functions that assign them meaning within a fuzzy system.

### Deriving terms
- There is no single correct set of terms; the number and shape of the terms is generally application dependent
- Many traditional mathematical functions can be used to describe the terms, but we are not limited to these. Could combine functions, enumerate values in $U$ and assign corresponding membership etc. 
- When **deriving terms** for a fuzzy system, questions may arise:
	- How many terms should there be?
	- What shape should they be?
	- How much overlap should there be between distinct terms?
- Some of the most common approaches to derive terms include:
	- **Conducting a survey** - e.g., asking many people what young means to them, then taking the average age range and combining the answers into a fuzzy set $M(young)$ either via enumeration or by designing a function to reflect the semantics correctly.
	- **Ask domain experts** - e.g., a doctor defines 'high-blood pressure' and 'low-blood pressure'
	- **Machine learning from data:** let an algorithm place and shape the terms to fit data/optimise performance, but oftentimes the resulting terms may no longer match what the words mean to people. 
	- **Trial and error**: adjust the terms until the system behaves well.
### Guidelines
- There are a number of heuristics that can be applied to membership functions of terms of a linguistic variable:
	- The term set should span $U$
		Every value should be described by at least one word, otherwise some inputs have no term at all, and a fuzzy rule system would have almost nothing to say about them. 
	- Terms shouldn't overlap too much
		- Two terms that almost completely overlap are barely distinguishable, so there is no point in having both; terms should be distinct in meaning.
	- Neighbouring terms should cross at around 0.5 membership
		- When plotted as a graph/have similar meaning, a value that is equally belonging to both terms should have 0.5 membership in both. 
	- Use a small number of terms $\leq 7$ 
	- All terms should be normal
	- All terms should be convex
	- Usually odd number of terms
- There are exceptions to these.

### Linguistic truth
- We now have the concept of a linguistic variable, and it is possible to have a *linguistic truth*:$$\chi = Truth$$
- $\chi$ = name of the linguistic variable e.g., height, temperature. 
- $U = [0,1]$, all possible degrees of truth
- $T(\chi) =$ 𝑡𝑟𝑢𝑒 + 𝑛𝑜𝑡 𝑡𝑟𝑢𝑒 + 𝑣𝑒𝑟𝑦 𝑡𝑟𝑢𝑒 + 𝑠𝑜𝑚𝑒𝑤ℎ𝑎𝑡 𝑡𝑟𝑢𝑒 + 𝑑𝑒𝑓𝑖𝑛𝑖𝑡𝑒𝑙𝑦 𝑡𝑟𝑢𝑒 + . . . + 𝑓𝑎𝑙𝑠𝑒 + 𝑛𝑜𝑡 𝑓𝑎𝑙𝑠𝑒 + 𝑣𝑒𝑟𝑦 𝑓𝑎𝑙𝑠𝑒 + 𝑠𝑜𝑚𝑒𝑤ℎ𝑎𝑡 𝑓𝑎𝑙𝑠𝑒 + 𝑑𝑒𝑓𝑖𝑛𝑖𝑡𝑒𝑙𝑦 𝑓𝑎𝑙𝑠𝑒 + . . .
- We can now represent and systematise statements such as 'that's not very true'.
	![](Pasted%20image%2020261010034925.png)
	OR alternatively:
	![](Pasted%20image%2020261010035105.png)
	1st number is degree of membership, 2nd is truth value
### Extension principle
- We have an ordinary function $f$ that works on single numbers, and we want to apply it to a *fuzzy input* such as 'small', and thus we need to define the result.
- Zadeh asserted a basic identity which allows a relationship from one domain to another to be extended into fuzzy domains
	$f$ is a mapping from $U$ to $V$ 
	$A = \mu_1 / u_1 + ... + \mu_n / u_n$ 
	Then
	$f(A) = f(\mu_1 / u_1 + ... + \mu_n / u_n) \equiv \mu_1 / f(u_1) + ... + \mu_n / f(u_n)$ 
- Apply $f$ to each element and keep each element's membership degree attached to the result.
	- If $u_1$ belonged to $A$ with degree $\mu_1$ then $f(u_1)$ belongs to $f(A)$ to that same degree.
	- It extends an ordinary function so it can take fuzzy sets as input.
	- The result(s) can live in a different universe $V$ which is why $f: U \to V$ 
		![](Pasted%20image%2020261010042840.png)





