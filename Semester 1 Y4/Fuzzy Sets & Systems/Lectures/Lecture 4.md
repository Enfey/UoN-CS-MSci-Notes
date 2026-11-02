## Tagaki Sugeno Kang(TSK) Fuzzy Inference
- Takagi, Sugeno and Kang introduced the second principal of fuzzy reasoning in 1985 and 1986
- It is quite similar to Mamdani inference, yet has no requirement for defuzzification, in that its output is immediately useful and is not a fuzzy output.
- It was also introduced in the context of fuzzy control
	- Thus, is based on control principles, rather than grounded in abstract logic
- It is now known as TSK inference
	- Broadly, it uses antecedents in the same way as in Mamdani inference, but uses a polynomial function for the consequent, which we will see shortly.

### Papers
- There are two foundational publications

#### Takagi and Sugeno - 1985 
- Central motivation was to develop a method for modelling complex systems using fuzzy descriptions of their behaviour
- A conventional mathematical model, as we have discussed before may try to model the system as one complex equation
- The alternative was to describe the system using a collection of *local models*
- Each model applies under particular conditions, expressed using fuzzy logic.
- For example:$$ R_1 : \text{IF } x \text{ is low} \\ \text{THEN } z = f_{1}(x, y)$$ $$ R_2 : \text{IF } x \text{ is low} \\ \text{THEN } z = f_{2}(x, y)$$
- Instead of describing the consequent as a linguistic variable, we describe it as a **linear polynomial function of the input variables**, which naturally come from the antecedents.
- By local model, we mean that each rule has its own mathematical model, or function, that predicts the output. 
	- We still aggregate such outputs
	- However, the functions for 2 different rules describing the same thing e.g., fan speed with different inputs will grow differently as the inputs change.
	- One model will describe how fan speed behaves at lower temperatures perhaps, while another describes the fan speed at higher temperatures.

#### Sugeno and Kang
- The 1986 paper applies fuzzy modelling and control to a multilayer incinerator
- TSK was motivated by the problem of modelling and controlling real systems, and was not a theoretical endeavour.
- The broader approach is to combine:
	- Fuzzy descriptions of operating conditions
	- Mathematical models describing system behaviour as inputs differ (consequents)
	- A mechanism for combining the models that apply. 
- A TSK rule may look like:
	*IF temperature is high THEN fan speed = 80*
- However, this is for crisp logic, for fuzzy logic this would look like:
	*IF temperature is high THEN fan speed = 2temperature + 10*
- We will see the more general form of how rules are defined in TSK inference shortly.
- We retain the fuzzy evaluation of the antecedents and merely change how we represent consequents in our rulebase, and how the final output is acquired.

## Regression analysis
- We introduce regression because first-order (fuzzy) TSK systems use linear functions in their consequents.
- **Regression analysis** is a statistical method to model the relationship between variables, based on **correlation**.
	- Imagine we have a list of houses
	- We have the size in *sq ft* and the sale price in *$* 
	- We want to understand the relationship between the size of a house, and its sale price
	- There appears to be a relationship; larger houses tend to sell for more money
		- This relationship is not perfectly consistent however.
- We can model such a relationship between variables, allowing us to estimate or predict one variable using information about another.
	- It mainly, for our purposes, shows how changes in one or more variables are aligned with changes in other variables.
- It involves 2 types of variables:
	- **Dependent variable** (or **regressand**)
		- What is to be estimated/predicted.
		- It is dependent on the independent variable/regressor; its value is modelled on the explanatory/regressor variable(s)
	- **Independent variable** (or **regressor**)
		- What is used to explain/predict the regressand.
- For our secnario, house price would be the regressand, and the regressor would be square feet, which aims to exxplain/predict the house price.

### Regression analysis 
- Suppose we plot our data set on a graph, with the x axis as the regressor, and the y axis as the regressand with each point representing a concrete observation
- Regression analysis seeks to 'draw' a line of best fit to that set/a certain set of observed data to describe the general relationship between the two variables for unseen instances of the independent variable.
	- We draw a 'line' because we assume that the relationship can be approximated by a linear function (which for our use case, it can)
	- This line will represent our model's predictions.
- It does so by minimising the distance between **actually observed values** and **estimated values**
#### The linear regression equation
- A linear regression equation captures the parameters of a linear regression model: $$\hat{Y} = \beta_{0} + \beta_{i}X_{i}$$
	Where:
	- $\hat{Y}$ is the estimated regressand
	- $\beta_0$ and $\beta_i$ are coefficients
	- $X_i$ is the instance of the regressor.
- More specifically, $\beta_0$ is the estimated average value of $Y$ when $X$ is zero/at the origin.
- $\beta_i$ is the estimated change in the value of $Y$ for one-unit change in $X_i$  
![](Pasted%20image%2020261101204152.png)

#### Where do we get our coefficients come from?
- We have our observed data, and we have the domain for the regressor variable.
- We do not know the correct values of $\beta_0$ and $\beta_1$ 
- We need to estimate them. 
- Regression finds the line that best fits the observed data by minimising the distance between actual and estimated values. 
- We call the difference between a concrete observation, and an estimated one for that same $X_i$ the **residual**:$$e_i = Y_i - \hat{Y_i}$$ 
- This tells us how far our prediction was from the actual observation
- We seek to produce a line with small residuals across all our observations. 
	- We often do this by the sum of squared residuals:$$\sum_{i=1}^n (Y_{i} - \hat{Y}_{i})^2$$
	- And this is what we seek to minimise(square because then every contribution is non-negative, prevent from cancelling out, and larger errors therefore receive disproportionately greater penalties)
- The regression algorithm should seek to find the coefficients that minimise this quantity.

**NOTE THAT EVERY OUTPUT OF REGRESSION EQUATION IS A PREDICTION/DESCRIBES A CORRELATION RELATIONSHIP AND IS NATURALLY NOT A GUARANTEE**

it is only as effective as the observed instances, and we cannot reasonably extrapolate far beyond the domain of the observed instances e.g., to mansions when we only have observed points for cottages, because the relationship may not hold.


### Types of linear regression and examples
- **Single-variable linear regression**
	- We have one regressand and one regressor
	- The equation remains as above: 
		- $\hat{Y} = \beta_0 + \beta_i X_i$
		![](Pasted%20image%2020261101205429.png)
- **Multi-variable linear regression**
	- One regressand and more than one regressor:
		$\hat{Y} = \beta_0 + \beta_1 X_1 + \dots + \beta_p X_p$
		Where $\beta_0$ is predicated on all $X$
### Zeroth order TSK
- The idea is that zeroth-order TSK replaces fuzzy output sets used in Mamdani with **numerical constants**
- These outputs from rules are then combined using a weighted average.
	- Naturally the weight comes from the firing strength obtained in the same way in Mamdani inference
- Rules of the following form are zeroth order:$$\begin{gathered} 
\text{IF } x \text{ is } A_{1} \text{ AND/OR } y \text{ is } B_{1} \dots \text{ THEN } z = k_{1} \\ 
\text{IF } x \text{ is } A_{2} \text{ AND/OR } y \text{ is } B_{2} \dots \text{ THEN } z = k_{2} \\ 
\vdots \\ 
\text{IF } x \text{ is } A_{i} \text{ AND/OR } y \text{ is } B_{i} \dots \text{ THEN } z = k_{i} \\ 
\end{gathered}$$
	Where:
	- $x$ and $y$ are crisp input values that relate to a linguistic variable e.g., temperature. 
	- $A$ and $B$ are fuzzy sets that denote the terms of their respective linguistic variables, used to directly determine how much rule $i$ applies.
	- Each $k_i$ for $i=1,...,n$ is a constant that defines the consequent.

### Generating an overall output
- The antecedents are evaluated the same as in Mamdani inference to determine the firing strength/how much the rule applies for each rule
- We denote this as $w_i$
- The weighted average of the $k_i$ associated with each rule gives the following output for zeroth order TSK systems:
	$$f_{TSK} = \frac{\sum_{i=0}^n w_{i}k_{i}}{\sum_{i=0}^n w_{i}}$$
- We must mutate the constant consequent by the firing strength of the rule, and then divide by the sum of the firing strengths
	- In dividing by the sum of firing strengths.. suppose we divided each firing strength by the summed total of the firing strengths
	- We technically normalise the firing strengths so that they all sum up to 1. 
	- So this formula technically converts each firing strength into its proportion of the total and then uses said proportions to calculate the output
	- Sure!


### Mamdani vs TSK example
![](Pasted%20image%2020261101212345.png)
- We have 3 rules:
	IF Age is young AND height is tall then Potential is 8 (good)
	IF Age is middle-aged THEN potential is 5 (fair)
	IF Age is old OR height is short THEN Potential is 2 (bad)
- We have the following inputs:
	- Age = 40(years)
	- Height = 1.8(metres)

#### Rule 1
- IF Age is young AND Height is tall then Potential is 8 (good)
- Antecedent 1: Age is young $\to \mu_{young}(40) = 0.25$
- Antecedent 2: Height is tall $\to \mu_{tall} = 0.75$
- Rule firing strength: $min(0.25, 0.75) = 0.25 = w_{1}$ 
- Consequent: Potential is good applies $k_1 = 8$ 
	- Note. We only weight them when we aggregate.

#### Rule 2
- IF Age is middle-aged THEN potential is 5 (fair)
- Antecedent 1: Age is middle-aged $\to \mu_{middle-aged}(40) = 0.5$ 
- Antecedent 2: n/a
- Rule firing strength: Antecedent 1
- $w_2 = 0.5$ 
- Consequent: potential is fair = 5 = $k_2$

#### Rule 3
- IF Age is old OR height is short THEN Potential is 2 (bad)
- Antecedent 1: Age is old $\to \mu_{old}(40) = 0$
- Antecedent 2: Height is short $\to \mu_{short}(1.8) = 0$
- Rule firing strength: 0, thus $w_3 = 0$ 
- Consequent: potential is bad = 2 = $k_3$

#### Rule combination/output
- We take the 3 rules outputs that applied (or in the case of rule 3, did not apply, but this will be reflected in the equation regardless)
	- $R1: w_1 = 0.25, k_1 = 8$
	- $R2: w_2 = 0.5, k_2 = 5$
	- $R3: w_3 = 0, k_3 = 2$ 
- We have the output function: $$f_{tsk} = \frac{\sum^3_{i=1}w_{i}k_{i}}{\sum_{i=1}^3 w_{i}}$$
- Output is thus: $$output = \frac{0.25 \times 8 + 0.5 \times 5 + 0 \times 2}{0.25 + 0.5 + 0} = 6$$

### Comparison of Mamdani and Sugeno
- We can see directly, the difference in how zeroth order TSK denotes its consequents compared to Mamdani inference
- We take a simple example: the *Tipper Example*
	IF Service is poor or Food is bad then Tip is cheap
	IF Service is good then Tip is average
	If Service is excellent or Food is delicious then Tip is generous
- We describe the consequent's linguistic variable Tip with the following triangular membership functions for its terms:
	![](Pasted%20image%2020261101215713.png)
- We describe the same consequent's linguistic variable by sampling the height of the fuzzy membership function, taking the domain value that yields maximal membership, and setting that as our constant
	- The idea is that the constant should be chosen such that it represents maximal membership in a fuzzy set
	- Thus weighting it by the firing strength of the antecedents will yield the same/similar shape as the membership function, but we do not have to perform defuzzification. 
	![](Pasted%20image%2020261101215936.png)

![](Pasted%20image%2020261101220059.png)
- Yielding roughly the same control surface.
- TSK can therefore approximate behaviour that would otherwise be modelled using Mamdani fuzzy output sets
	- Naturally, they need not be the same
	- The shapes will depend on the membership functions, consequent definitions, and the defuzzification method. 

### Comparison of the computational operations involved in Mamdani and TSK inference

|                            |                                                                                Mamdani                                                                                 |                                       TSK                                        |
| :------------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------: | :------------------------------------------------------------------------------: |
| **Antecedents**            |                                                                      Antecedent eval is the same                                                                       |                           Antecedent eval is the same                            |
| **Rule evaluation**        |                                               Operations across consequent set depending on degree/amount of antecedents                                               | Single multiplication of $w_i$ against the consequent (or a few for first-order) |
| **Rule aggregation**       | Aggregation operates across each rule, usually picking $max$ as the aggregation operator for discrete instances of $y$ with relation to the constrained fuzzy outputs. |                                    Summation                                     |
| **Defuzzification/output** |                                      Method depends entirely, but generally more complicated than simple summation and division.                                       |                      Summation per rule and Single division                      |
- This comparison depends entirely on the defuzzification method.
	- For example, we may use the **Centre-of-Sums** defuzzification method; the most commonly used  defuzzification method which is extremely fast.
	- It converts fuzzy set into a single, crisp output value by calculating a weighted average of the individual centroids of each output fuzzy set
	- Instead of taking the algebraic union via rule aggregation(which yields an irregular composite shape) as input, we instead look at each output fuzzy set ($C_i$) separately
	- For each scaled output membership function $C_i$, we find:
		- The individual area $A_i$ 
		- The geometric center or centroid $x_i$ 
	- The crisp output value $y^*$ is computed as:$$y^* = \frac{\sum_{i=1}^n x_{i} \cdot A_{i}}{\sum_{i=1}^n A_{i}}$$
	- CoS counts overlapping areas of distinct fuzzy output sets multiple times, whereas they are only counted once for ordinary centroid method given that the output set has been derived via the max operator applied per $y$
	- Now, all of this is relevant, because we make the pointwise max aggregation step disappear; we do not need to apply it for $n$ constrained output sets over $U$ universe
	- We only need to calculate the area and centroid of $n$ output sets, and can ignore $U$ entirely. 
	- CoS is not computationally equivalent to TSK, but it shows that we can forego some computation by clever choice of defuzzification method.
- This comparison also depends on the output/consequent universe of discourse, as this will change for example, for how many points we have to consider when performing rule aggregation in Mamdani inference.
- IT also affects our rule evaluation: at the fuzzy set level we apply the min operator to the entire consequent membership function with our firing strength; this is applied pointwise, naturally, and thus the size of the domain directly affects how many times we perform *min* pointwise, for $n$ rules.
- Furthermore, computational performance is not the only consideration:
	- Accuracy - how well does the model represent the underlying system?
	- Interpretability/explainability - Mamdani's linguistic output sets and their relation to the derived crisp output value  can be easier for humans to understand rather than dealing with discrete values.
	- Upfront design complexity - it may be more difficult to design good membership functions for the consequent's linguistic variable terms, than it would be to define a constant that performs 'well-enough'
	- Verifiability - how easily can we inspect, and reason about the systems behaviour?


### First Order TSK
- In First-Order TSK, each rule produces a linear polynomial function of the inputs as a consequent.
- This is in contrast to the zeroth order TSK where each rule produces a constant that is then weighted.
- The general form of rules in a first-order TSK system are as follows:$$\begin{gathered}
\text{IF } x_{1} \text{ is } A_{1} \text{ AND } y \text{ is } B_{1} \text{ THEN } z \text{ is } p_{1}x+q_{1}y+r_{1} \\

\text{IF } x_{2} \text{ is } A_{2} \text{ AND } y \text{ is } B_{2} \text{ THEN } z \text{ is } p_{2}x+q_{2}y+r_{2} \\

\vdots \\

\text{IF } x_{n} \text{ is } A_{n} \text{ AND } y \text{ is } B_{n} \text{ THEN } z \text{ is } p_{n}x+q_{n}y+r_{n} \\

\end{gathered}$$
	Where:
	- $A$ and $B$ denote fuzzy sets/terms that describe the linguistic variable that the variables $x$ and $y$ are representative of
	- $x_{i}$ and $y_{i}$ are inputs
	- $p_i, q_i$ and $r_i$ are **rule-specific** constants for $i=1, ... n$ 
- We can see that this models the consequent as a polynomial function of the inputs $x$ and $y$ with the function $f$ differing for each rule $R$.
- Each rule's proposed output changes as the input changes:
	- Recall that $\hat{Y} = \beta_0 + \beta_1 X_1 + ... + \beta_p X_p$ 
	- Compare this with the first-order consequent: $p_i x_i + q_i y_i + r_i$ 
	- They have the same mathematical structure.
	- Essentially, each rule has its own regression equation:
		- The input variables $x$ and $y$ are multiplied by the rule-specific coefficients, and the intercept $\beta_0$ that appears in the regression equation is merely $r_i$
	- This doesn't tell us anything beyond that they have the same structure, or how to pick the coefficients for each rule. 
	- In a data-driven approach, we may collect observations of inputs and outputs.
	- We could then construct a rulebase that approximates this relationship, with the consequent being defined by performing this regression analysis/optimising some function that represents the least amount of difference between observed and predicted values
		- We might identify a number of fuzzy regions e.g., small houses, relatively new & large houses, relatively new and give each region its own first-order TSK consequent. 
		- Deciding the rules themselves is a bit more tricky, we often use a combination of domain knowledge, optimisation to minimise prediction error for given fuzzy regions/relationships, and derivation from data.
	- This greatly increases the **accuracy** of our fuzzy system; we cannot choose the coefficients arbitrarily, unless we have an existing point of reference and just tune toward that. 
#### Simple example
- We refer back to the tipper example.
- We define the following first order rules:
	- If Service is poor or Food is bad then Tip = service + food + 5
	- If Service is good then Tip = service + food + 15
	- If Service is excellent or Food is delicious then Tip = service + food + 25
- Each rule's consequent is predicated on the inputs to the rule, with coefficients that are defined separately.
	- E.g., we could define the food coefficient in $R2$ to be $0$ because food is not an antecedent linguistic variable. 
- As with all TSK systems, we combine the rule outputs using their firing strengths.
- We calculate the rules discrete value first.
- To keep it simple, we will have $p$ and $q$ = 1
- Generating the final output is as follows:$$y^* = \frac{\sum_{i=1}^3w_{i}z_{i}}{\sum_{i=1}^3w_{i}}$$
- The general structure of individual rule output for rule $i$ is as follows:$$z_{i} = \sum_{j=1}^n p_{ij}x_{i}+r_{i}$$
#### Example output surfaces
![](Pasted%20image%2020261101232116.png)


### Higher order consequents
- TSK consequents do not have to be restricted to constants or first-order linear functions
- We have seen two orders: zeroth and first.
- **Higher order** TSK could use higher-order polynomial consequents, defined like so: $$ z_i = a_{i}x^2 + b_{i}y^2 + c_{i}xy + d_{i}x + e_{i}y + r_{i}$$
- This gives each rule a more expressive local model, and can represent relationships that a purely linear function cannot capture. 
- Higher-order polynomials such as these have been used to describe consequents in extremely specific circumstances, as well as other non-linear functions, but these are rarely used. 
- More expressive consequents mean there are more parameters to tune/optimise for generally
	- A more complicated model is generally harder to tune, requires more data to determine coefficients/parameters, and harms the interpretability and explainability of the model
	- Also increases upfront design cost; if it can be done with a simpler model, generally it should.
### Tuning TSK systems
- There has been substantial research into the tuning of TSK systems, where tuning emans adjusting the parameters of the fuzzy system to improve how accurately it models the desired input-output relationship across the domain. 
- There are several kinds of parameters we could adjust:
	- **Antecedent parameters** - the shapes and positions of membership functions, which determine how strongly rules fire.
	- **Consequent parameters** - the coefficients in each rule's output function e.g., $p_i, q_i, r_i$ 
	- **Rule structure** - the number of rules, and the way that the input space/identified fuzzy regions are divided amongst the rule space to achieve the desired outcome. 

#### Combining TSK systems with neural networks
- Neural networks can be used to help learn of optimise parameters of a TSK system
	- E.g., adjust membership functions, consequent coefficients, to reduction prediction error according to some function e.g., square of least errors (with respect to the residuals)

### Pros and Cons of TSK vs Mamdani
- **Design Complexity**
	- **Mamdani**: 
		- Linguistic rules relatively intuitive to construct - consequent is also linguistic and requires much less design than in TSK, with a few terms and membership functions covering most rules.
		- Defuzzification method choice does factor in here; they are not interchangeable, and will have impact on performance and computational efficiency.
	- **TSK**
		- Depends on the order, generally consequent functions and parameters, can be considerably more complex.
		- Rulebase design is often more complex; we don't have linguistic variable terms as consequents, we have to design our consequents ourselves, for every single rule.
		- Choice of order also factors, and the choice of *what* to optimise, if the relationship we are trying to emulate is non-linear becomes more complex.
- **Performance**
	- We refer to the ability to **model relationships here**
	- **Mamdani**
		- Can model complex relationships; suitability depends on the application
	- **TSK**
		- Also capable of modelling complex numerical relationships, especially as one can increase the order of the system. 
- **Tuning/optimisation**
	- **Mamdani**
		- Can be tuned too, but its linguistic structure and prevalence of fuzzy output and fuzzy structures may require a different approach e.g., tuning the output membership functions themselves. 
	- **TSK**
		- Particularly amenable to numerical optimisation and learning
- **Adaptation**
	- **Mamdani**
		- Adaptation is possible, but will generally involve modifying membership functions and rule structure by hand, is much more tedious. 
	- **TSK**
		- Numerical parameters can be adjusted to model different relationships or a change in the domain with new examples, relatively easily.
			- E.g., give it the new observation points and retrain to optimise the coefficients for the new domain (but naturally have the same variables/inputs in this case)
- **Interpretability**
	- **Mamdani**
		- Linguistic rules are often easier for humans, and non-specialists to understand on a day-to-day basis
	- **TSK**
		- Mathematical consequents may be less intuitive to non-specialists, and if any portion is tuned automatically, the system may be *harder to explain*.
- **Interaction**
	- **Mamdani**
		- Exposes linguistic rules, which offer a natural basis for human facing explanations without dealing with complex mathematical functions/models
		- Can easily be visualised.
	- **TSK**
		- Can visualise the local regression model each consequent describes; can expose the numerical relationships directly at finer granularity.
- **Verifiability/Accountability**
	- **Mamdani**
		- Linguistic structure can make decisions easier to inspect and communicate.
	- **TSK**
		- Mathematical structure can support formal analysis



