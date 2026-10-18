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
	- In classicial logic, we would determine membership of inputs for the respective terms of the lingustic variables.
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
- Note that any given proposition we represent in fuzzy logic represents merely a term, not a linguistic variable itself(unless we compact into the linguistic variable beforehand with additional logic such as set intersection etc), and so our universe of discourse may not necessarily grow, it depends entirely on the logical relationship between sets that we are trying to represent.


### Generalised Modus Ponens

#### If...Then rules in Fuzzy Systems

###