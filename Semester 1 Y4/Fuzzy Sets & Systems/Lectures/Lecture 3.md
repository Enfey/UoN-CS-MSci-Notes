### Combining logic and sets
- Consider a tautology:
	- $(p \to q) \iff \neg(p \wedge(\neg q))$
		- $(p \to q) \iff \neg p \vee q$ 
- Tautologies let us link **crisp sets** and logic by expressing implication e.g., $p \to q$ 
- Suppose we are designing a system that decides how many layers of clothing someone should wear based on the temperature
- We maintain a rule:
	*IF the temperature is cold, THEN the number of clothing layers should be many*
	Our linguistic variables naturally being temperature and clothing layers. 
- We have an overarching logical implication.
	- In classical logic, we would determine membership of inputs for the respective terms of the linguistic variables.
	- But the result would be binary.
- We can express this better with fuzzy logic:
	$\mu_{p \to q}(x,y) = 1-\mu_{p\cap \overline{q}}(x,y) = 1 - min(\mu_{p}(x), 1-\mu_{q}(y))$
	OR alternatively, for the solved variant:
		$\mu_{p\rightarrow q}(x,y)=\max(1-\mu_p(x),\mu_q(y))$
	This denotes logical implication of fuzzy sets.
- We now deal with 2 different universes:
	- $X$: possible temperatures
	- $Y$: possible numbers of clothing layers
	- Our implication thus constructs a fuzzy relation $\mu_R: X \times Y \to \{0,1\}$ 
- E.g., we put in 5 degrees, and 1 layer of clothing and get $\mu_{p\to q}$= 0.1 which suggests that this combination does not strongly agree with the implication we are representing. 
- A proposition in fuzzy logic expresses a linguistic term, with multiple propositions potentially being combined using fuzzy logical operators to produce a composite fuzzy set or concept e.g., very warm may not be directly defined, scale membership of top-$n$ valid domain members.
	- 'Compacting' terms into a proposition like this may not necessarily enlarge the universe of discourse though; the combination changes their membership function but retains the original domain. 

### Modus Ponens
- **Modus Ponens** is a **rule of inference** in classical logic:
$$ p \to q$$
$$p$$
$$\therefore q$$
- **Premise 1**: If $p$, then $q$ (conditional, implication)
- **Premise 2**: $p$ is $TRUE$ (antecedent)
- **Conclusion:** Therefore, $q$ is true. (consequent)


### Modus tollens
- Modus tollens reverses the direction of the inference by using the negation of the conclusion:
$$p \to q$$
$$ \neg q$$
$$\therefore \neg p$$
- Denies the consequent $q$, rather than asserting the antecedent $p$ 

### Generalising logic
- Classical logic represents propositions as either true or false, with no regard for representing partial truths/memberships, that we are concerned with in fuzzy logic. 
- We want to extend the above inference rules to fuzzy logic, so we can reason about fuzzy sets in a similar manner. 

#### Generalised Modus Ponens
- Generalised modus ponens changes the second premise so that it no longer has to match the rule's antecedent exactly
- The structure becomes the following:
$$\text{If $x$ is $A$, then $y$ is $B$}$$
$$x \ is \ A^*$$
$$\therefore y \ is \ B^*$$
- Here, $A$ and $B$ are fuzzy sets representing original concepts/terms of a linguistic variables, while $A^*$ and $B^*$ denote the observed condition and inferred conclusion. 
- The asterisks denote that these fuzzy sets need not be identical to the original antecedent $A$ or consequent $B$ in the defined rule.
- Note: $\mu_{A \to B}(x,y) = 1 - min(\mu_A(x), 1-\mu_B(y))$ where $\mu(x) : X \to [0,1]$ 
	- For example, **if it is cold, people wear many layers of clothing.**
	- Suppose that $\mu_A(x) = 0.8$ and $\mu_B(y) = 0.3$ 
	- We generalise the above statement, as we have a different proposition: the temperature $x$ is $(5^\circ C)$, rather than simply asserting that $x$ is cold much like we would in crisp logic.
	- Fuzzy inference allows us to determine how closely this **observation** matches the antecedent **Cold** via its degree of membership, and use this match to infer a conclusion about the consequent **Many Layers**.
	- The inferred consequent $(B^*)$ will *depend on the inference method* used to combine the observation $(A^*)$ with the fuzzy implication relation $(A \rightarrow B)$ 
### If...Then rules in Fuzzy Systems
- Inference is performed by utilising a set of rules connecting premises to conclusions.
- A fuzzy inference system contains rules of the form: $IF \ x \ is \ A$, $THEN \ y \ is \ B$.
- The process of using an **observation** and the rule to derive a conclusion is called ***inference***.
	- We are deriving the conclusion $B^*$, as this needs to be derived in relation to our observation, and the rule we are applying. 
	- Calculating the fuzzy relation i.e., $\mu_{A \rightarrow B}$ only tells us for a given pair of observations how well the rule is satisfied.
	- We're interested in taking the initial observation and deriving a conclusion, where the rule applies.
### Establishing the degree of truth of an antecedent
![](Pasted%20image%2020261023225650.png)
Naturally this is not universal; the membership function for the consequent does not have sloping edges, nor does it need to, so it translates 1:1 with crisp logic. 

### Example
![](Pasted%20image%2020261023225836.png)
- We have an antecedent, and are interested in applying fuzzy inference to determine the consequent in the application of the rule to said antecedent.
- For a rule with multiple antecedents, the **firing strength** is usually calculated by combining their membership degrees with a fuzzy AND operator such as the minimum or product.
- Several rules may contribute to a conclusion e.g.,:
	- *IF it is raining, THEN it is cloudy.*
	- *IF humidity is high, THEN it is cloudy.*
	- *IF atmospheric pressure is low, THEN it is cloudy*
- Their outputs can be aggregated to determine the overall inferred fuzzy set for cloudy given the antecedents.
- Practical fuzzy inference systems often use a **rule base** rather than a single rule.

#### Quick note on fuzzy logic
##### Law of excluded middle
- In classical logic a prop must be true or false
- The law of excluded middle states:
$$A \cup \overline{A} = X$$
- Where $X$ is the universe of discourse.
- Using the standard fuzzy complement and maximum-based union:
	- $\mu_{\overline{A}}(x) = 1 - \mu_{A}(x)$
	- $\mu_{\overline{A} \cup A}(x) = max(\mu_A(x), 1-\mu_A(x))$ 
	- Suppose an apple has membership 0.6 in Red. 
	- Its membership in Not Red is 0.4, via the negation.
	- However, their union has membership 0.6, not 1(it should be 1 for every element in the universe, thus is proof by contradiction), and so this law does not hold under the established fuzzy operators that satisfy the required axioms. 
##### Law of contradiction
- In classical/crisp logic, a proposition and its negation cannot both be true:$$A \cap \overline{A} = \emptyset$$
- WIth minimum based intersection:
	- $\mu_{A \cap \overline{A}}(x) = min(\mu_A(x), 1-\mu_A(x))$ 
- For our apple, this would yield 0.4; the intersection has nonzero membership. 
- Other fuzzy operators may behave differently, but its enough of a case that both of these laws generally do not apply to fuzzy logic.


### Fuzzy Inference - Applications
- Used for controlling real-world systems where inputs are uncertain, behaviour is complicated, or engineers already have useful rules expressed in natural language.
- Traditional control systems use mathematical models and precisely defined thresholds
- In comparison, fuzzy controllers encode expert knowledge as rules, evaluate sensor measurements/inputs against fuzzy concepts, and combine rules to determine a control action. 
	- For example, rather than requiring an engineer to specify an exact threshold for a high temperature, a controller can represent temperature as low, medium, high, very high etc with overlapping membership functions for more adaptable control outcomes. 
- For example, fuzzy control was used in train operation in the Sendai Railway in Japan, including acceleration and braking.
- Instead of treating every driving situation as a simple fixed threshold or binary, the controller could combine several conditions with differing degrees and determine an appropriate control response. 
	- Fuzzy logic, as we'll get onto later, helps handle vagueness and uncertainty in measurements
	- It does not automatically eliminate sensor noise or guarantee better control outcomes than conventional controllers, but it offers a higher degree of adaptability and granularity for certain control scenarios.

## Mamdani inference
- Mamdani introduced the first successful form of fuzzy inferencing in 1975
- The original paper investigated whether heuristic rules supplied by a human operator could be translated into an automatic controller.
- The experiment used a model industrial plan consisting of a steam engine. 
### Methodology
- **For each rule**:
	- For each antecedent in a given rule, evaluate the membership degree of the crisp input in the fuzzy term (*Fuzzification*, $\mu_{A_1}(x_1)$)
	- Combine the antecedents membership degrees according to the natural language that appears in the rule e.g., IF it is *cold* and *humid*, use a *t-norm*
	- Fire the rule at the strength (combined degree) to obtain the rule output based on the rule's consequent. 
- **Aggregate** the output of each rule relating to the consequent that has fired, and interpret in some way (e.g., defuzzify the fuzzy output to a crisp value)

#### Rule firing and rule strength
- Consider a fuzzy rule:
	- IF the temperature is *cold*, THEN fan speed is *low*.
- Suppose the temperature sensor measures 5 Degrees Celcius, and when we fuzzify this input we get back $\mu_{Cold}(x) = 0.8$
- Since the rule only has one antecedent, the **firing strength** is simply:$$\alpha = 0.8$$
- What we mean when we say that a rule *fires* is that we evaluate the model antecedent against our antecedent(s) strength.
	- Consider 0 degrees, with $\alpha =1.0$, this rule fires fully, as the input has membership 1 in cold; this naturally works the opposite way.
- We represent Low as a fuzzy set modelling possible fan speeds.
- Suppose we have $\mu_{Low}(y)$, the rule's firing strength modifies this consequent using the implication method
- Commonly, in Mamdani inference, we *clip* or *constrain* the consequent fuzzy set according to the strength of the antecedent:$$\mu'_{Low}(y) = min (0.8, \mu_{Low}(y))$$
- In other words, no point in the rule's output fuzzy set can have membership greater than 0.8; affecting how much the rule applies.
	- Naturally, we do not have a particular $y$ after firing a rule, we just get a constrained fuzzy set as an intermediate result.
- For rules with multiple antecedents, we take the strength of the antecedent by applying the necessary fuzzy operators e.g., t-norms, t-conorms, complement, to derive the firing strength to constrain our resulting consequent fuzzy set membership values.

#### Aggregating the outputs of multiple rules
- If we have multiple rules that contribute to the same fuzzy consequent, then we fire them simultaneously providing we have the inputs.
- Suppose we have a fan controller with these two rules:
	RULE 1: IF temperature is *cold*, THEN fan speed is *low*
	RULE 2: IF temperature is *hot*, THEN fan speed is *high*
- For some particular measured temperatures, fuzzified, suppose their firing strengths are $\alpha_1 = 0.7$, $\alpha_2 = 0.6$ 
- These rules fire to the respective degrees, minimum clipping the fuzzy output with respect to the strength of the antecedents.
- We have two separate output fuzzy sets, both describing the same linguistic variable: fan speed.
- We need to combine them into a single fuzzy set before we can produce one final crisp fan-speed value.
- In standard Mamdani inference, a common aggregation operator is maximum:
	- Let two rule outputs be $\mu'_1(y)$ and $\mu'_2(y)$ over constrained fuzzy sets. Then:
		- $\mu_{aggregate}(y) = max(\mu'_{1}(y), \mu'_{2}(y))$
	- We calculate this for every single possible valid output/domain value $y$. 
	- E.g., ![](Pasted%20image%2020261024011350.png)


### Defuzzification - briefly.
- Now we have the aggregated fuzzy set, but it is still not a single value(this will depend on what the application needs - if one just needs a membership value for given input $y$ then we can stop without performing this step).
- We'll go with the fan speed example from above.
- One common method is the **centroid method**:$$y^* = \frac{\int_{Y} y \ \mu_{agg}(y) \ dy}{\int_{Y} \mu_{agg}(y) \ dy} $$
	Accumulating with respect to the variable $y$, that is, the output/concrete value.
	Accumulate all membership values across the output universe, geometrically yielding the area under the consequent/aggregated consequent membership function. 
	Accumulate all membership degrees according to the corresponding output value $y$; but multiple each membership degree by its corresponding output value $y$ before adding the contributions. 
	The result will be a a crisp output $y^*$, the centre of area of the aggregated fuzzy set,
Similarly:
![](Pasted%20image%2020261024013009.png)
![](Pasted%20image%2020261024013012.png)
For discrete; often what we deal with. 


We're TECHNICALLY DONE FROM THIS POINT ON, BUT I LEARNED FROM ANOTHER SOURCE. BELOW COVERS THE LECTURE SLIDES AS THEY APPEAR, RATHER THAN MY OWN TERMINOLOGY, JUST SO IM FAMILIAR WITH BOTH.
### Determining the degree of truth of an antecedent (should have thought this was obvious, just covering because it is on the slides)
- Given a numeric input $x'$, for example age $x' = 40$ 
- We convert the numeric input into a fuzzy set (not a typo) using a process called ***fuzzification***
- **Fuzzification:** 
	- Here we use **singleton fuzzification** i.e., we denote a fuzzy set whose membership values are crisp: $0$ for all $x$ $\neq 40$, and $1$ for $x = 40$:
$$\mu_{X'}(x) = 1 \ if \ x = x',\ 0 \ otherwise$$
- This connects to the notion of an antecedent:
	- Consider a fuzzy set *Middle-Aged*, that has the membership function $\mu_A(x)$ 
	- The intersection of the singleton observation fuzzy set $X'$ and the fuzzy set $A$ can be denoted as:$$\mu_{X' \cap A}(x) = min(\mu_{X'}(x), \mu_{A}(x))$$
	- At every $x \neq 40$, the singleton membership is zero.
	- At $x = 40$: $$\mu_{X' \cap A}(40) = min(1, \mu_{A}(40))$$
	- Therefore, intersecting the singleton observation with the antecedent fuzzy set and taking the maximum membership(for singleton, this is just $\mu_A(x)$) gives you the antecedent's membership degree at the observed input.
	- Just take $\mu_A(x)$ ffs, taking max of min under these assumptions is redundant, basically just ignore this slide when revising; can just calculate membership directly. 
### Multiple antecedents example
![](Pasted%20image%2020261024014416.png)

### Firing a consequent of a rule example
- We model the firing strength as a a **cylindrical extension** $C$
- Compute the output set as the intersection of $C$ and the consequent
- In this example, the consequent is good:
	![](Pasted%20image%2020261024014907.png)
	Just clip to 0.25 via intersection:
	$\mu_{Rule \ 1}(p) = min(\mu_{C}(p), \mu_{Good}(p))$, where $\mu_C(p)$ is just the constant maximum membership level of 0.25 for all potentials across the output domain.
	Thus we can interpret as:$\mu_{Rule_{1}}(p) = min(0.25, \mu_{Good}(p))$ 
![](Pasted%20image%2020261024015211.png)


### Aggregating the outputs of multiple rules example
- Outputs from individual rules are combined using disjunction i.e., the union of both rules' outputs, operationalised using a t-conorm, typically *maximum*
- We did indeed do this earlier:
	- Let two rule outputs be $\mu'_1(y)$ and $\mu'_2(y)$ over constrained fuzzy sets. Then:
		- $\mu_{aggregate}(y) = max(\mu'_{1}(y), \mu'_{2}(y))$
- We'll see it in the lecture style here:
	- Constrain the fuzzy sets individually via intersection with the cylindrical extension $C$ derived from the rule firing strength $\alpha$ 
	- Then perform the union of these constrained fuzzy sets for the given output values $y$ to produce the aggregate fuzzy set.
![](Pasted%20image%2020261024015527.png)


### Common Operators
- Mamdani inference features union and intersection operators, in multiple places
	- Intersection combining antecedents joined by AND
	- Union combining antecedents joined by OR
	- Intersection to produce the consequent fuzzy set from the original with the cylindrical extension $C$.
	- Union combining consequent fuzzy sets (with implication/clipping applied)
- Operator families should be used consistently in practice, often AND-OR pair is varied independently of implication and combination?
- How we operationalise the t-norms and t-conorms in practice is application dependent, we will see a variety of operators in future.

### Defuzzification
- The process of converting the a fuzzy set to a number is a process called 'Defuzzification'
- Strictly speaking, the result of a mamdani inference is a complex output.
	- It is a distribution, and more specifically, the membership function characterising the (potentially aggregated) consequent fuzzy set
- To attain a useful, crisp output from this distribution/fuzzy set, there are multiple defuzzification approaches we can take, broadly falling into two principal approaches:
	1. **Numeric defuzzification**
		- Converts the output fuzzy set to a single crisp number
		- For example, a constrained, aggregated fuzzy set that is the output of mamdani inference may be defuzzified to yield a crisp output of **65**
		- There are different methods of extracting such a number.
	2. **Linguistic defuzzification**
		- This method of defuzzification choose a linguistic term that best represents the output fuzzy set such as *Low*, *Medium*, or *High*.
		- The idea is to compare the output set with predefined fuzzy sets representing known linguistic terms using a similarity or distance measure.
#### Numeric Defuzzification
##### Centroid Defuzzification (mentioned in lecture)
- The **centroid** method finds the centre of area of the aggregated membership function
- If we think of the membership graph as a flat, irregular shape, we seek to find where this would **balance** if it were a physical object.
- If our output universe is continuous, the formula is:$$y^* = \frac{\int_{Y} y \ \mu_{agg}(y) \ dy}{\int_{Y} \mu_{agg}(y) \ dy} $$
	The denominator is merely the total area under the membership function. 
	The numerator weights the area by the output value(s) $y$(s).
	Dividing them gives the centre of area.
- Note, we're generally dealing with discrete universes in the context of this course, so we use the formula with sums: $$y^* = \frac{\sum_{i}y_{i}\mu_{agg}(y_{i})}{\sum_{i}\mu_{agg}(y_{i})}$$
	![](Pasted%20image%2020261024024247.png)

##### Mean of Maxima (MoM) (mentioned in lecture)
- The mean-of-maxima method ignores most of the membership function
- It observes output values that have the **highest membership degree**, then takers their arithmetic mean:$$x_{MoM} = mean(argmax_{X}\mu(x))$$
- So if the aggregated set is clipped to 0.25, and it occurs for 4 values, simply add the 4 values, divide by 4, and you have $y^*_{MoM}$.

##### Visualisation of centroid and MoM
![](Pasted%20image%2020261024024531.png)


##### Height defuzzification
- This is a defuzzification method which works according to individual rule contributions
- Taking the $\alpha$ of each rule determined by the antecedents, and a representative consequent value $c_i$, we can define this formula
- Note that this formula mathematically does not require multiple rule contributions, but this is where its weighted combination becomes useful. 
	- E.g., multiple rules with different consequents, can avoid aggregation/union of the consequents and just do this directly, as opposed to say aggregation and then performing centroid or mean of maxima with respect to the aggregated membership function. 
$$y^* = \frac{\sum_{i}a_{i}c_{i}}{\sum_{i}a_{i}}$$
	![](Pasted%20image%2020261024025119.png)
	Maybe come back to this.
- ***Estimate output by taking weighted average of representative values from individual rule contributions.*** 
##### Centre-of-sets defuzzification
- Exactly the same as above, but $c_i$ is generally the centroid of each consequent fuzzy set as opposed to the domain element with maximal height.
##### Bisector defuzzification
- Finds the output value that divides the area under the aggregated membership function into two equal halves
- If the total area is $A$, the chosen output $y^*$ satisfies:$$\text{Area to the left of $y^* = $} \frac{A}{2}$$
	and:$$\text{Area to the right of $y^* =$} \frac{A}{2}$$
	The idea is to find the point where the cumulative membership weight (not scaled) reaches half the total. 