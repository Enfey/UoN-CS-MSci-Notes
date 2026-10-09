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
	- **Axiom c3** - **Continuity**
		$c$ should be a continuous function. A tiny change in membership should only cause a tiny change in the complement. 
	- **Axiom c4** - **Involution**
		- $c$ should be involute, that is $c(c(a)) = a, \forall a \in [0,1]$ 
- All fuzzy complements satisfy the required axioms $c1$ and $c2$.
- Continuous complements are complements that further satisfy $c3$.
- All functions satisfying axioms $c4$ and $c2$, form a further nested sub-class of complements as they are automatically continuous. 


