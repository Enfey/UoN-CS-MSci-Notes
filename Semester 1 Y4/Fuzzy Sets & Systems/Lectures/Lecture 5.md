## Philosophical motivation of Fuzzy logics
- Bertrand Russel argued that classical logic presumes precise symbols, which real life does not supply
- Everything is vague to a degree we don't notice until we try to make it precise e.g., cheese, speaking generally, is a dairy product, but how much of that instance of cheese or type of cheese is made precisely of dairy?
	- The more specific we get, and the less we think generally, the harder it is to map to crisp logics.
- Black made a similar point about measurement; indeterminacy is not just a quirk of everyday language, it is present in all scientific m,easurement
- Vagueness is a basic property then of how we interpret and describe the world.
	![](Pasted%20image%2020261109230211.png)
### Practical motivation of Fuzzy logic
- Zadeh's "computing with words" (1996) observes that people park cars, drive in traffic, play golf, understand speech and summarise stories without explicit measurement and computation. 
- They participate in activities and discuss them by manipulating what he called *perceptions* : size, distance, speed, likelihood, weight, time and other metrics were described in terms of natural language such as "fairly close" or "quite fast"
- A machine that wants to represent such metrics needs formalisms that can hold that kind of statement
	- And thus aids the explainability of a system, as this is how we naturally describe notions of precision and measurement in everyday life.

### Summary of Motivations
- The world we reside is imprecise and vague
- We understand that world, and experts make decisions that influence the world with imprecise data and language
- They work **successfully** with knowledge that is rarely defined *mathematically* or *algorithmically* but uses vague, perhaps ambiguous terminology with words
- Type-1 Fuzzy sets express a **degree** of uncertainty, and faithfully denote the notion that the world is not black and white, and represents vague measurements and membership
	- However, ***what if we are uncertain about that degree of uncertainty/membership?***

#### Example
- Say $22^{\circ}C$ is "warm" to degree $0.7$
- That $0.7$ is a perfectly precise number that someone had to choose (via designing some membership function)
- Several things make this shaky:
	- Different people, geographies, and cultures mean different things by "warm"
	- The sensor that supplies $x$ may be noisy
	- The data used to derive the membership function is itself noisy.
- Where we describe vagueness with a perfectly crisp measurement, we have just pushed the precision problem up one level and abstracted it away, without removing it. 
- Membership grades themselves must then, become fuzzy.
- This is a *Type-2 Fuzzy Set*

## Type-n Fuzzy Sets
- A fuzzy set is of type $n$, $n \in \{2,3,4, ...\}$ if its membership function ranges over fuzzy sets of type $n-1$ (similar to currying)
- The membership function of a type-1 ranges over the interval $[0,1]$ and yields this as a result, as opposed to yielding a type $n-1$ fuzzy set; it does not recurse.

## Type-2 Fuzzy Sets
- **Type-1 Fuzzy set A**
	- $A = \{(x, \mu_A(x) \ \vert \ \forall x \in X\} \equiv \int_{x \in X} \mu_A / x$
		- For a discrete universe $X_d$, $A = \sum_{x \in X_d} \mu_A(x) / x$ 
		- Note that $\mu_A(x) / x$ is not a division (its a label)
		- $\int$ and $\sum$ are not used arithmetically either.
		E.g.,
		$A = \{(20, 0.2), (22, 0.7), (25, 1)\}$ and $A = 0.2/20 + 0.7/22 + 1/25$ are equivalent.
- **General Type-2 Fuzzy set $\tilde{A}$**
	- $\tilde{A} = \{((x, u), \mu_{\tilde{A}}(x,u)) \ \vert \ \forall x \in X, \forall u \in J_x \subseteq [0,1] \}$ 
		- Where $x$ is an element of universe of discourse $X$
		- Where $u$ is a candidate membership degree for $x$, and $u \in J_x$ which defines a number of candidate membership degrees for $x$
			- This specifies a range of of possible memberships
			- We can specify the entire range of $[0,1]$
		- Note that nothing computes $u$ directly, from say $x$, it is defined outside of this, and they become arguments to the inner type-1 fuzzy membership function. 
- **Interval Type-2 fuzzy set $\tilde{A}$**
	- $\tilde{A} ={((x, u), 1) \ \vert \ \forall x \in X, \forall u \in J_x \subseteq [0,1]}$
		- The structure is identical, with the only difference being the grade function
		- The third component of the type-2 set says nothing.
		- Every candidate is simply 'possible' to equal degree
		- Only $J_x$ carries information by specifying the candidate degrees of membership as a range.
- **Modern Notation**
	- $\tilde{A} : X \to F([0,1])$
	- $x \to \tilde{A}(x) \equiv A_x$ 
	- Where:
		- $A_x : [0,1] \to [0,1] : u_x \to A_x(u_x)$ 
	We can read this as the universe of discourse's elements mapping to some function over [0,1] where this function is element-wise, and is a membership function defining a type-1 fuzzy set for that element, which takes candidate membership degrees $u_x$ and returns a values in $[0,1]$ to determine the plausibility of $x$ and that membership degree.
The type-1 set $A_x$ is attached to $x$, and it takes a range of candidate membeship values for $x$ as input and yields the plausibility of that pair. 


**Each element $x$ of the universe $X$ is mapped to its own type-1 fuzzy set $A_x$ over $[0,1]$. Its membership function takes a candidate membership degree $u$ and returns a value in $[0,1]$ saying how plausible it is that $u$ is the membership degree of $x$.**
	**As such, the grades of memberships are themselves fuzzy.**

Always say **plausible**, not likelihood.

### Operations
#### Union
- The union of two type-2 fuzzy sets $\tilde{A}$ and $\tilde{B}$ is defined below:$$\tilde{A} \cup \tilde{B} \iff \mu_{\tilde{A} \cup \tilde{B}} = \mu_{\tilde{A}(x)} \ \sqcup \mu_{\tilde{B}(x)} = \sum_{i}\sum_{j}(f(u_{i}) \ \star \ g(w_{j})) /(u_{i} \vee w_{j})$$
- Here, $f = A_x$ and $g = B_x$ 
- And $\star$ is a **tnorm**, $\vee$ is maximum, and $\sum$ is a collection operator.
- Once more, the slash is not division.
- Example:
	- At some $x$, with $A_x = \{0.4 : 1, 0.8 : 0.5\}$ and $B_x\{0.3 : 0.2, 0.6 : 1\}$
	- Thus we have:
		- Grade $min$: 
			- $min(1, 0.2) = 0.2$ 
			- $min(1, 1) = 1$
			- $min(0.5, 0.2) = 0.2$
			- $min(0.5, 1) = 0.5$
		- Candidate $max$:
			- $max(0.4, 0.3) = 0.4$
			- $max(0.4, 0.6) = 0.6$
			- $max(0.8, 0.3) = 0.8$
			- $max(0.8, 0.6) = 0.8$
		- Thus $(\tilde{A} \cup \tilde{B})_x = \{0.4 : 0.2, 0.6 : 1 , 0.8 : 0.5\}$
			(because we see 0.8 twice, we just take the maximum of the grades)(also note that this is reversed to have the candidate membership values first, as it is typically notated, but is not defined as such)
- Taking the union of two type-2 sets means combining two uncertain membership degrees into one uncertain degree for "A or B"
- At each $x$ you apply the ordinary type-1 union to each pair of candidates the 2 sets allow.
- We score the pairs by taking the min of the plausibilities of the elements of the pair.
- If we have a duplicate candidate value, we take the max of the grade/plausibility of that candidate value in the new fuzzy set. 

#### Intersection
- The intersection of two type-2 fuzzy sets is given by:$$\tilde{A} \cap \tilde{B} \iff \mu_{\tilde{A} \cap \tilde{B}}(x) = \mu_{\tilde{A}}(x) \sqcap \mu_{\tilde{B}}(x) = \sum_{i}\sum_{j} ( f(u_i) \star g(w_j))  (u_i \wedge w_j)$$
- Here $f = A_x$ and $g = B_x$ 
- And $\star$ is a **tnorm**, $\wedge$ is minmum, and $\sum$ is a collection operator.
- Only two symbols change $\sqcup$ becomes $\sqcap$ (the meet, instead of the join) and we use min for the candidate membership values $u_i$ and $w_j$ of $A_x$ and $B_x$.
	- Naturally we do so because we are performing the intersection element wise of the inner type-1 fuzzy sets. 

WE can perhaps think of these operations as operating on vertical slices of the respective type-2 sets; taking one slice each, which represents a type-1 fuzzy set over some $x$ and then returning a single slice defined by the operator to construct the new type-2 fuzzy set, for all $x$. 

We use the t-norm in both because even though we pick a 'winning' candidacy value for $x$ at that point in the type-1 set, if we only picked the plausibility of whichever candidacy value won of either $u_i$ or $w_j$, we would not be representing any information about its plausibility (which is still obviously has) in the other set; we'd just be discarding that information. Taking the min tells you conservatively how plausible that value is across both sets, while still preserving the type-1 definition of the intersection/union operation. 

SURE!!!!
### Structure of system
![](Pasted%20image%2020261110014611.png)
- The **Fuzzifier** turns a crisp input $x'$ into type-2 input sets. 
	- Each linguistic term(say, warm or cold) is its own type-2 set over domain $X$
	- For a given $x \in X$, each term has exactly one slice in that set: a type-1 set over candidate degrees, saying how plausible each degree is for $x$ belonging to that term. 
	- Singleton input:
		- Fuzzifier turns the crisp input $x'$ into a type-2 fuzzy set $I$ defined on $X$, with $\mu_I(x) = 1$ if $x = x'$, and 0 otherwise
	- Non-singleton input:
		- The membership function for the type-2 fuzzy set defined for the crisp input differs.
		- For $\mu_I(x) = 1$, if $x = x'$, and falls off as $x$ moves away from $x'$.
		- This is commonly a gaussian membership function. 
- Covered next lecture
### Why use/not use Type-2
#### Use
- **Fine-grained uncertainty modelling**
	- At each $x$, the secondary membership function $A_x$ describes the uncertainty about the primary membership grade. 
	- We can define different types of type 2 e.g., general and interval to model real world scenarios where the degree of uncertainty of membership will fluctuate.
- **Strong in domains where the degree of uncertainty varies**
	- For example in outdoor robotics where sensors are affected by changing wind conditions
	- In a non-singleton system, the sensor noise is the $\sigma$ of the input set's membership function. 
#### Not use
- **Computationally complex**
	- Every single simple operation does considerably more work due to the nesting and amount of type-1 fuzzy sets over which the operations are defined
	- Type-2 output sets of the inference engine must also be **type-reduced** to type-1 fuzzy output sets, which is a the most expensive step.
- **Effort**
	- You have to define an inner membership function for every $x$, and nothing tells you how to choose them
	- Without data to define these membership functions or expert knowledge, choices become hard to justify. 
### Representation
- **Embedded sets**
	- An **embedded set** of a type-2 set picks exactly one candidate membership grade $u \in J_x$, for every $x$.
	- For a discrete domain $X_d = \{x_1, ..., x_N\}$ with $J_{x_i}$, we describe a triple of $\{x_i, \phi(x_{i}), z_{i}\}$
		- Where $\phi(x_i)$ represents a selection of from $J_{x_i}$ (who denotes the domain of the inner membership function)
		- Where $z$ represents $A_{x_i}(\phi(x_i))$ - the plausibility of the chosen candidate membership value. 
	- We do this for all domain elements $x_1 \dots x_N$ , taking one triple each for each $x$.
	- $N$ triples thus forms a singular embedded set.
	- The number of an embedded sets is $|J_{x_1}| \times |J_{x_2}| \times |J_{x_3}| \times \dots \times |J_{x_n}|$ 
	![](Pasted%20image%2020261110031828.png)
	- Mendel and John's representation theorem states that a type-2 fuzzy set can be represented as the *union* of its embedded sets
		- Triples of dimension $N$
			- Merge duplcates, pool all triples = get back the exact same type-2 set
- This theorem is useful because it reduces type-2 questions to type-1 questions?
- However, we have $|J_{x_1}| \times |J_{x_2}| \times |J_{x_3}| \times \dots \times |J_{x_N}|$ embedded sets. With $N$ domain points and let us say $M$ candidates each($\prod_i M_i$), that is $M^N$.
- The count grows exponentially in $N$, so even for a very small number of discretisations (replacing continuous domain $X$ with finite list of points, and giving continuous $J_x$ finite list of $M$ candidate values) it becomes very difficult to enumerate all embedded sets.

### Interval type-2 Fuzzy sets.
- An interval type-2 fuzzy set sets $A_x(u) =1$ for $u \in J_x$, and $0$ otherwise
- The secondary membership function now behaves like crisp set membership
- We have what we call **upper** and **lower** membership functions(MF) for the the range that $J_x$ describes
	- $\overline{\mu} = Upper MF$ 
	- $\underline{\mu} = LowerMF$ 
	![](Pasted%20image%2020261110044337.png)
	Where $x = 21$ 
	The shaded pattern is $J_{21}$, the candidacy degrees of membership for the type-2 fuzzy set 'Generous' whose secondary membership function is fixed at 1. 
	Thus describes the interval between the upper and lower MF's (and these upper and lower MFs implicitly describe the slice itself, as everything in between has secondary membership 1)
- In a general type-2 set, the union at each $x$ needs the double sum over every pair of primary grades, with a t-norm on the secondary grades. 
- In an interval set, every vertical slice has the secondary grade 1, so we can completely cut out not only computing the membership, but also taking the min, and its inclusion in the summation, greatly reducing the computational complexity.
	- Type reduction becomes less complex and becomes solvable with iterative approaches and approximations.
- For example, we can compute the union for two interval type-2 fuzzy sets: $$\tilde{A} \cup \tilde{B} = 1/FOU(\tilde{A} \cup \tilde{B}) = 1/[\underline{\mu}_{\tilde{A}}(x) \vee \underline{\mu}_{\tilde{B}}(x),\overline{\mu}_{\tilde{A}}(x) \vee \overline{\mu}_{\tilde{B}}(x) ]$$
	- In other words the result is once more, an interval type-2 set described by taking the pointwise max of the lower and upper MF's respectively.
	- Because the upper and lower MF implicitly describe the set of candidacy values with no variation in the secondary membership function output, this is valid. 
		- This simplifies operations on type-2 fuzzy sets significantly.


### zSlices
- A $zSlice$ can be thought of like so:
	- At each $x$, keep only the primary grades whose secondary grades are at least $z$: $$\tilde{A}_{z} = \{(x, u) : A_{x}(u) \geq z\}$$
	- Each slice then, is an interval type-2 set, because everything that survives the cut is treated as having one grade; the secondary membership function for the primary grades that survive the cut is crisp.
- We choose some $z$ s.t. $0 \lt z \leq 1$
- A general type-2 set then, is represented as a set of slices/IT2 at levels $0 \lt z_1 \lt z_2 \lt ... \lt z_X$.
	- Note we fix a given $z$ across all $x$. 
	- If some $x$ never reaches $z$, then the slice is simply empty at that $x$
- Because each slice is an IT2, we can use everything we described prior; type-reduction becomes easier, union, other operations, and should we convert the entire general type-2 set to zSlices, we... 


## Comparison between General Type-2, Interval Type-2, Zlices

## K-Means Clustering

## Fuzzy c-means

### Fuzzy c-means vs K-means

