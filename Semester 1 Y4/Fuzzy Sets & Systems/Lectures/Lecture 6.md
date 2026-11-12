## Interpretable AI (XAI)
- XAI *normally* stands for **explainable AI**
- We use the the two terms interchangeably, but we can define what we mean concretely below.
	- **Miller** - Interpretability is the *degree to which an observer can understand the cause of a decision.* It depends on who the observer is, and it is about the **cause** of a particular decision.
	- **Biran and Cotton**: Systems are interpretable if their operations can be understood by a human; this is either through introspection/reasoning (read the model itself), or a produced system explanation. 
- For example, a machine learning system which takes images as input and classifies animals, recognises a cat. 
- Just saying 'cat' does not explain how the AI arrived at this decision. 
- An XAI explanation would return the label, plus its justification: whiskers, fur, claws, tail, body length, etc.
	The system is thus explainable/interpretable in some manner, as there is a produced system explanation.
	![](Pasted%20image%2020261111210451.png)
### XAI - Fuzzy sets and systems
- We're interested in whether fuzzy systems are interpretable. 
- We break this down into components; Fuzzy sets, Rulebase, Operators, Structure(e.g., hierarchical fuzzy sets?).
- Some of these **can be** interpretable i.e., understood by *some* humans, subject to certain criteria e.g., do the FSs reflect a linguistic meaning, is there a reasonable amount of rules, and do they map to real-world concepts.
	- Thus the answer is conditional. 
- The criteria we have matches two standard categories in the fuzzy interpretability literature:
	- **Semantic Interpretability** - do the fuzzy sets reflect a *linguistic meaning*? Typical conditions are that the sets: 
		- Distinguishable from one another
		- Cover the domain 
		- Are consistent with their labels (and this is agreed upon)
	- **Complexity** - how many rules, how many antecedents per rule, and how many sets do describe each linguistic variable, defuzzification method, etc.
#### Our focus
- We focus on a small part of interpretability of fuzzy logic systems
- We focus on the **systematic modelling of input uncertainty**
	- We want to maximise our capacity to explain and communicate **why** and **how** fuzzy systems address and handle *input uncertainty.*
	- We want to tune only the related parts of the system of a non-uncertain input system (singleton) to allow the system to map cleanly to uncertain inputs, and that these design choices are explicit, effective, and overtly use available information, best practices, and insight
	- This allows us to justify our ability to model this input uncertainty, and allows us to explain how outcomes are achieved in the system with respect to these modifications. 

## Non-singleton fuzzy logic systems
- In **non-singleton fuzzy systems** (NSFLs), we do not use a singleton fuzzy set to model our input/domain, but a *non-singleton* fuzzy set.
	- We will explain this below
- The non-singleton fuzzy set captures that we have **some** knowledge about the input $x$.
### Fuzzification
- **Singleton fuzzification**
	- We use a *singleton fuzzy set to model our input*
	- What we mean by this is we have a reading $x'$, and a domain value $x$.
	- The membership function $\mu_I(x) = 1$ if $x = x'$, $0$ otherwise.
	- This is our singleton fuzzy set predicated on $x$ for a given input $x'$; naturally it is a crisp singleton set (which is just a special case of fuzzy set)
- **Non-singleton fuzzification**
	- We use a *non-singleton fuzzy set to model our input*
	- What we mean by this is instead of giving membership only where $x' = x$, the non-singleton input set gives graded membership values around domain values similar to $x'$
	- Say we had the reading $x' = 22$, the membership function for the input fuzzy set may be something like:
		$$\mu_{I}(x) = max (0, 1-\frac{x-22}{2})$$
		Every value between 20 and 24 is a member, with membership falling as you move away from 22, yielding a genuine type-1 fuzzy set.
- The non-singleton fuzzy set captures that we have *some* knowledge about the actual value $x'$ that we read.

![](Pasted%20image%2020261111214909.png)

### Epistemic and Ontic NS Fuzzy sets
- **Epistemic**
	- An epistemic fuzzy set **represents incomplete knowledge** about a single true value.
	- It captures a *disjunctive relation* between elements i.e., one of the elements (values on $x$) is the correct input, but we do not know which one
		- The membership grades naturally denote how plausible each domain element is as being the correct value.
	- It captures that we have some incomplete knowledge of the actual value $x'$
	- We represent all the knowledge we are privy to in this epistemic set and this forms our input NSFS for our NSFLs
	- The example before was an example of an epistemic system:
		- The membership unction expresses that the value is probably near 22, but we have incomplete knowledge e.g., sensor noise. 
- **Ontic**
	- The set itself is the actual value, and does not model uncertainty.
	- It captures a *conjunctive* relationship between its elements, in that they are all belong together and jointly make up the input.
	- Nothing here is unknown; we are not modelling uncertainty and have complete knowledge of the input.
	- For example, a patient's blood pressure ranged from 110-140 over a day. 
		- This interval is the observation itself, and every value in it occurred.
			- (the input can be the interval itself e.g., pass output of an algorithm directly to NSFLS, or pass FS by representing as FS and then destructuring it? i suppose!)
			- the same interval is epistemic if it describes a range of possible readings rather than 'all the values occurred.'
		- Thus all have membership degree of $1$ in an ontic fuzzy set as the knowledge is 'complete'.

### Distribution
- **Data-driven**
	- Is the input set's membership function designed according to observed data e.g., the spread of recent readings?
- **A priori**
	- Is the input set's membership function designed from knowledge known before any data is seen e.g., a sensors specified accuracy of $\pm 0.5^{\circ}$?
	- Can be either a priori or data-driven for epistemic; ontic is not designed a priori as the set is just the datum. 
- **Normal**
	- Does at least one value $x$ have maximal membership (1) in the input fuzzy set?
	- For an epistemic set, this is expected, since the true value lies in the domain. 
	- An ontic set need not be normal e.g., the membership values may be derived according to the mode and mean, which may not yield an $x$ with $\mu_I(x) = 1$ 
- **Convex**
	- Does the membership function have no dips i.e., never decrease then increase again?
	- Possible for both convex and epistemic.
	- Triangles and gaussians are convex, the shape below is not:
		![](Pasted%20image%2020261111221136.png)

### Motivation - Modelling uncerainty where it occurs

### Fuzzification and Firing Strength - Challenges

#### NSFLs - Standard Composition

#### NSFLs - Inference alternatives

##### Does the choice matter?

##### Impact/Performance

### Adaptive NSFLs