# Conversational AI architectures
- Typical pipeline:
	1. **<mark style="background: #FFF3A3A6;">Preprocessing</mark>**
		Tokenisation, normalisation
	2. **<mark style="background: #FFF3A3A6;">Representation</mark>**
		BOW
	3. **<mark style="background: #FFF3A3A6;">NLU</mark>**
		Intent, extract meaning
	4. <mark style="background: #FFF3A3A6;">Dialogue management</mark>
		<mark style="background: #FFF3A3A6;">State</mark>, <mark style="background: #FFF3A3A6;">entities</mark>, <mark style="background: #FFF3A3A6;">system calls</mark>
	5. **<mark style="background: #FFF3A3A6;">NLG</mark>**
		<mark style="background: #FFF3A3A6;">Produce</mark> <mark style="background: #FFF3A3A6;">final</mark> <mark style="background: #FFF3A3A6;">response</mark>
Useful agent must:
- Track **<mark style="background: #FFF3A3A6;">state</mark>**
- Use **<mark style="background: #FFF3A3A6;">memory</mark>**
- Interact with **<mark style="background: #FFF3A3A6;">external systems</mark>**
- **<mark style="background: #FFF3A3A6;">Learn/improve</mark>** over time

## Architectures for Conversational Agents
### Rule-Based Systems
- <mark style="background: #FFF3A3A6;">Trad paradigm</mark>, <mark style="background: #FFF3A3A6;">pattern matching</mark>, <mark style="background: #FFF3A3A6;">hand-written rules</mark>
- <mark style="background: #FFF3A3A6;">Dialogue manager</mark> = <mark style="background: #FFF3A3A6;">state machine</mark>
- <mark style="background: #FFF3A3A6;">Predictable,</mark> <mark style="background: #FFF3A3A6;">reliable</mark> <mark style="background: #FFF3A3A6;">interpretable</mark>
- <mark style="background: #FFF3A3A6;">Not robust</mark>, <mark style="background: #FFF3A3A6;">not scalable</mark>
### Retrieval-based systems
- <mark style="background: #FFF3A3A6;">Instead of gen text</mark>, <mark style="background: #FFF3A3A6;">select response</mark>,<mark style="background: #FFF3A3A6;"> existing corpus</mark>
- <mark style="background: #FFF3A3A6;">Types of retrieval based systems</mark> include
	- <mark style="background: #FFF3A3A6;">Simple matching</mark> e.g., <mark style="background: #FFF3A3A6;">BOW</mark>
	- <mark style="background: #FFF3A3A6;">Neural retrieval modes</mark>
- <mark style="background: #FFF3A3A6;">Response quality good</mark>, <mark style="background: #FFF3A3A6;">grounding</mark>
- <mark style="background: #FFF3A3A6;">No novelty</mark>, <mark style="background: #FFF3A3A6;">not flexible</mark>, <mark style="background: #FFF3A3A6;">lack context sensitivity </mark>w/o pre-processing

### Generative models
- Use <mark style="background: #FFF3A3A6;">neural language models</mark>, <mark style="background: #FFF3A3A6;">produce response</mark>
- Evolved over time
	- RNNs, LSTMs
	- Transformers
- <mark style="background: #FFF3A3A6;">Flexible</mark>, <mark style="background: #FFF3A3A6;">context awareness</mark>
- Cons: <mark style="background: #FFF3A3A6;">Not grounded</mark>, <mark style="background: #FFF3A3A6;">inconsistent,</mark> need <mark style="background: #FFF3A3A6;">signif data.</mark>

### Hybrid Systems - Retrieval Augmented Generation
- <mark style="background: #FFF3A3A6;">Combination</mark>, <mark style="background: #FFF3A3A6;">retrieval-based</mark>, <mark style="background: #FFF3A3A6;">generative models</mark>, <mark style="background: #FFF3A3A6;">provide grounding</mark>, <mark style="background: #FFF3A3A6;">generative models</mark>. 
	![[Pasted image 20251119212133.png]]
	<mark style="background: #FFF3A3A6;">USE QUERY</mark> TO <mark style="background: #FFF3A3A6;">GET ENHANCED CONTEXT</mark>, <mark style="background: #FFF3A3A6;">ATTACH TO OG INPUT</mark>, THEN <mark style="background: #FFF3A3A6;">RETURN RESPONSE</mark>.
- <mark style="background: #FFF3A3A6;">Contextual</mark>, <mark style="background: #FFF3A3A6;">grounded</mark>, <mark style="background: #FFF3A3A6;">flexible</mark>
- Inconsistent, <mark style="background: #FFF3A3A6;">data intensive</mark>, <mark style="background: #FFF3A3A6;">hard implement</mark>

### Hybrid Systems - Cognitive Architectures
- <mark style="background: #FFF3A3A6;">Advanced</mark>, '<mark style="background: #FFF3A3A6;">agent framework</mark>', <mark style="background: #FFF3A3A6;">reason,</mark><mark style="background: #FFF3A3A6;"> plan</mark>, <mark style="background: #FFF3A3A6;">memory</mark>, <mark style="background: #FFF3A3A6;">tool use</mark>, interact w/systems
	![[Pasted image 20251119212315.png]]
- <mark style="background: #FFF3A3A6;">Flexible</mark>, <mark style="background: #FFF3A3A6;">context-aware</mark>, <mark style="background: #FFF3A3A6;">grounded</mark>, <mark style="background: #FFF3A3A6;">interact with system</mark>
- <mark style="background: #FFF3A3A6;">Data intensive</mark>,<mark style="background: #FFF3A3A6;"> hard implement</mark>, <mark style="background: #FFF3A3A6;">consistency</mark>

# Human in the Loop AI Systems
- <mark style="background: #FFF3A3A6;">Trad ML</mark>, <mark style="background: #FFF3A3A6;">focus,</mark> <mark style="background: #FFF3A3A6;">full automation</mark>, has limits:
	- <mark style="background: #FFF3A3A6;">Models, saturate</mark>, <mark style="background: #FFF3A3A6;">benchmark datasets</mark>
	- <mark style="background: #FFF3A3A6;">Overspecialise</mark>
	- Suffer from "**<mark style="background: #FFF3A3A6;">concept drift</mark>**"
		- <mark style="background: #FFF3A3A6;">Statistical properties</mark> of <mark style="background: #FFF3A3A6;">target var </mark><mark style="background: #FFF3A3A6;">change over time</mark> in unforeseen ways
	- <mark style="background: #FFF3A3A6;">Cannot adapt efficiently</mark> due to <mark style="background: #FFF3A3A6;">overspecialisation</mark>
	- <mark style="background: #FFF3A3A6;">Unsafe</mark> without oversight
- <mark style="background: #FFF3A3A6;">Human in loop</mark>; participate in:
	- <mark style="background: #FFF3A3A6;">Training</mark>
	- <mark style="background: #FFF3A3A6;">Tuning</mark>
	- <mark style="background: #FFF3A3A6;">Labelling</mark>
	- <mark style="background: #FFF3A3A6;">Correcting</mark>
	- <mark style="background: #FFF3A3A6;">Evaluating</mark>
	-<mark style="background: #FFF3A3A6;"> Interpreting</mark>
## Interpretability
- <mark style="background: #FFF3A3A6;">Interpretability</mark> of<mark style="background: #FFF3A3A6;"> model</mark> + <mark style="background: #FFF3A3A6;">results,</mark> <mark style="background: #FFF3A3A6;">build</mark> **<mark style="background: #FFF3A3A6;">trust</mark>** (via how mistakes handled, not success), enables:
	- <mark style="background: #FFF3A3A6;">Sanity-check</mark> results
	- <mark style="background: #FFF3A3A6;"> Accountability</mark>
	- <mark style="background: #FFF3A3A6;">Debugging </mark>inference process
	- <mark style="background: #FFF3A3A6;">Understanding model errors</mark>
- However, <mark style="background: #FFF3A3A6;">what does interpretable mean</mark>?
- Interpretable by whom?
- And, at what cost?
### Taxonomy of interpretability
#### Intrinsic vs Post-Hoc
- **<mark style="background: #FFF3A3A6;">Intrinsic</mark>** - <mark style="background: #FFF3A3A6;">interpretability</mark> = <mark style="background: #FFF3A3A6;">property of model</mark>
- **<mark style="background: #FFF3A3A6;">Post-hoc</mark>** - <mark style="background: #FFF3A3A6;">interpret predictions</mark> <mark style="background: #FFF3A3A6;">after the fact</mark>
#### Model-specific vs Model-agnostic
- **<mark style="background: #FFF3A3A6;">Model-specific</mark>** - <mark style="background: #FFF3A3A6;">interpretation methods</mark> <mark style="background: #FFF3A3A6;">tied</mark> to <mark style="background: #FFF3A3A6;">particular models</mark>
- **<mark style="background: #FFF3A3A6;">Model-agnostic</mark>** - <mark style="background: #FFF3A3A6;">interpretation methods</mark> <mark style="background: #FFF3A3A6;">applicable</mark> to <mark style="background: #FFF3A3A6;">any model</mark>
#### Local vs Global
- **<mark style="background: #FFF3A3A6;">Local</mark>** - <mark style="background: #FFF3A3A6;">interpret</mark> <mark style="background: #FFF3A3A6;">one prediction</mark> <mark style="background: #FFF3A3A6;">at a time</mark>
- **<mark style="background: #FFF3A3A6;">Global</mark>** - <mark style="background: #FFF3A3A6;">interpret</mark> the <mark style="background: #FFF3A3A6;">entire model</mark>

### Examples of Intrinsically Interpretable Models
- **<mark style="background: #FFF3A3A6;">Decision trees</mark>**
	- <mark style="background: #FFF3A3A6;">Model classification</mark>, <mark style="background: #FFF3A3A6;">finite sequence</mark>, <mark style="background: #FFF3A3A6;">discrete decisions</mark>
	- <mark style="background: #FFF3A3A6;">Interpret easily</mark> until tree, reach, explosive depth
	- <mark style="background: #FFF3A3A6;">Learned through algos</mark> <mark style="background: #FFF3A3A6;">seek minimise</mark> <mark style="background: #FFF3A3A6;">info</mark>rmation <mark style="background: #FFF3A3A6;">entropy</mark>
- **<mark style="background: #FFF3A3A6;">KNN</mark>**
	- <mark style="background: #FFF3A3A6;">Classifies</mark> <mark style="background: #FFF3A3A6;">based on distances</mark>
	- <mark style="background: #FFF3A3A6;">Interpretability</mark> <mark style="background: #FFF3A3A6;">trivial </mark>in <mark style="background: #FFF3A3A6;">low-dimensional spaces</mark>.

### Model-Agnostic Methods
- <mark style="background: #FFF3A3A6;">Interpretation methods</mark>, **<mark style="background: #FFF3A3A6;">applicable</mark>, <mark style="background: #FFF3A3A6;">any model</mark>**; separation of explanation from model
- <mark style="background: #FFF3A3A6;">Desirable properties,</mark> model-agnostic explanation system
	- **<mark style="background: #FFF3A3A6;">Model flexibility</mark>**
		- Work w/ any model
	- **<mark style="background: #FFF3A3A6;">Explanation flexibility</mark>**
		- Multiple types of explanation
	- **<mark style="background: #FFF3A3A6;">Representation flexibility</mark>**
		- Map explanation, differing types, representations
#### Global Surrogate models
- <mark style="background: #FFF3A3A6;">Train</mark>, <mark style="background: #FFF3A3A6;">interpretable model</mark> e.g.,<mark style="background: #FFF3A3A6;"> decision tree</mark>, <mark style="background: #FFF3A3A6;">approximate</mark>, decisions, **<mark style="background: #FFF3A3A6;">black box model</mark>**
- E.g., <mark style="background: #FFF3A3A6;">train SVM </mark>on $(X,Y)$
- <mark style="background: #FFF3A3A6;">Classify dataset</mark>, giving $(X, \hat{Y}$)
- <mark style="background: #FFF3A3A6;">Train decision tree</mark> on  $(X, \hat{Y})$
- <mark style="background: #FFF3A3A6;">If accuracy of decision tree acceptable</mark>, <mark style="background: #FFF3A3A6;">use</mark> to <mark style="background: #FFF3A3A6;">explain decisions of SVM</mark>
- Approximate entire black box.
#### Local Surrogate models
- <mark style="background: #FFF3A3A6;">Used</mark>, <mark style="background: #FFF3A3A6;">explain</mark>, <mark style="background: #FFF3A3A6;">individual predictions</mark>, black box machine learning models. 
	- <mark style="background: #FFF3A3A6;">Why did model</mark>,<mark style="background: #FFF3A3A6;"> make</mark>, <mark style="background: #FFF3A3A6;">specific prediction</mark>?
	- <mark style="background: #FFF3A3A6;">What effect,</mark> did, <mark style="background: #FFF3A3A6;">feature val</mark>, <mark style="background: #FFF3A3A6;">have on prediction</mark>?
- <mark style="background: #FFF3A3A6;">Use simple</mark>,<mark style="background: #FFF3A3A6;"> interpretable model</mark>,<mark style="background: #FFF3A3A6;"> trained</mark>, <mark style="background: #FFF3A3A6;">approximate black-box model</mark>, in <mark style="background: #FFF3A3A6;">small neighbourhood</mark><mark style="background: #FFF3A3A6;"> around specific prediction</mark>. 
- <mark style="background: #FFF3A3A6;">Best known</mark> = **LIME - "<mark style="background: #FFF3A3A6;">Locally Interpretable Model Estimation</mark>"**
	- <mark style="background: #FFF3A3A6;">Generate</mark>, <mark style="background: #FFF3A3A6;">perturbed versions</mark>, <mark style="background: #FFF3A3A6;">one example</mark>; synthetic data points
	- <mark style="background: #FFF3A3A6;">For each synthetic point</mark>, <mark style="background: #FFF3A3A6;">feed</mark> back <mark style="background: #FFF3A3A6;">into black box</mark>,<mark style="background: #FFF3A3A6;"> record predictions</mark>
	- <mark style="background: #FFF3A3A6;">Weight samples</mark> via <mark style="background: #FFF3A3A6;">proximity</mark>; <mark style="background: #FFF3A3A6;">distant points</mark> may f<mark style="background: #FFF3A3A6;">ollow different decision logic</mark>
	- <mark style="background: #FFF3A3A6;">Train linear</mark>, <mark style="background: #FFF3A3A6;">interpretable model</mark>, on <mark style="background: #FFF3A3A6;">weighted</mark>, <mark style="background: #FFF3A3A6;">perturbed dataset</mark>
	- <mark style="background: #FFF3A3A6;">Determine</mark>:
		- <mark style="background: #FFF3A3A6;">Which features</mark> <mark style="background: #FFF3A3A6;">contributed</mark> <mark style="background: #FFF3A3A6;">most </mark>to <mark style="background: #FFF3A3A6;">prediction</mark>
		- <mark style="background: #FFF3A3A6;">Whether</mark> they <mark style="background: #FFF3A3A6;">pushed prediction</mark>, <mark style="background: #FFF3A3A6;">toward/away</mark> <mark style="background: #FFF3A3A6;">class</mark>
		- <mark style="background: #FFF3A3A6;">How sensitive</mark>, <mark style="background: #FFF3A3A6;">prediction is</mark>, <mark style="background: #FFF3A3A6;">feature changes.</mark>
	- **<mark style="background: #FFF3A3A6;">Feature-based explanation</mark>**, <mark style="background: #FFF3A3A6;">decomposing predictions</mark> into <mark style="background: #FFF3A3A6;">feature effects;</mark> vectors not indivisible.


#### Explaining with examples
- **<mark style="background: #FFF3A3A6;">Model agnostic technique</mark>**; <mark style="background: #FFF3A3A6;">interpret bb model</mark>, <mark style="background: #FFF3A3A6;">present</mark>ing, <mark style="background: #FFF3A3A6;">concrete instances</mark>, <mark style="background: #FFF3A3A6;">rather than,</mark> model params, <mark style="background: #FFF3A3A6;">feature weights</mark>, or rules. 
- Provide more intuitive, holistic focus
- <mark style="background: #FFF3A3A6;">Examples show</mark>:
	- <mark style="background: #FFF3A3A6;">Implicit decision boundary</mark>(<mark style="background: #FFF3A3A6;">adversarial examples</mark>, <mark style="background: #FFF3A3A6;">counterfactual explanation</mark>)
	- <mark style="background: #FFF3A3A6;">Representative instances</mark>(**prototypes**), and <mark style="background: #FFF3A3A6;">unrepresented examples</mark>(**criticisms**)
	- Which **<mark style="background: #FFF3A3A6;">instances</mark>**, have <mark style="background: #FFF3A3A6;">highest influence</mark>, <mark style="background: #FFF3A3A6;">model parameters.</mark>

#### Counterfactual explanation
- <mark style="background: #FFF3A3A6;">Describe</mark>, <mark style="background: #FFF3A3A6;">closest alternative</mark> <mark style="background: #FFF3A3A6;">instance</mark>, <mark style="background: #FFF3A3A6;">original input</mark> , that <mark style="background: #FFF3A3A6;">would result</mark>, **<mark style="background: #FFF3A3A6;">different prediction</mark>**
- <mark style="background: #FFF3A3A6;">Explains</mark>, <mark style="background: #FFF3A3A6;">individual predictions</mark>, used, <mark style="background: #FFF3A3A6;">identify</mark>,<mark style="background: #FFF3A3A6;"> causal links</mark> <mark style="background: #FFF3A3A6;">between</mark> <mark style="background: #FFF3A3A6;">concrete instances</mark>
- "If feature x was higher by y, prediction would have been positive"
	- <mark style="background: #FFF3A3A6;">Interpretable</mark>
	- <mark style="background: #FFF3A3A6;">Flexible</mark>
	- <mark style="background: #FFF3A3A6;">Easy to implement</mark>, <mark style="background: #FFF3A3A6;">slightly perturb</mark>, <mark style="background: #FFF3A3A6;">until prediction changes</mark>, <mark style="background: #FFF3A3A6;">select closest valid alternative</mark> to propagate counterfactually
	- <mark style="background: #FFF3A3A6;">Often multiple counterfactual explanations</mark>, for <mark style="background: #FFF3A3A6;">single outcome. </mark>
#### Prototypes
- **<mark style="background: #FFF3A3A6;">Prototypes</mark>** = **<mark style="background: #FFF3A3A6;">representative instances</mark>** of <mark style="background: #FFF3A3A6;">class</mark>, <mark style="background: #FFF3A3A6;">capture</mark>, what model/data, considers *<mark style="background: #FFF3A3A6;">typical</mark>*
- **<mark style="background: #FFF3A3A6;">Criticism</mark>** = <mark style="background: #FFF3A3A6;">instance</mark>, **<mark style="background: #FFF3A3A6;">not well represented</mark>**, <mark style="background: #FFF3A3A6;">by prototypes</mark>.
- Prototypes, <mark style="background: #FFF3A3A6;">can be selected manually</mark>, <mark style="background: #FFF3A3A6;">does not scale well,</mark> poor results; <mark style="background: #FFF3A3A6;">many approaches to finding</mark>, <mark style="background: #FFF3A3A6;">any clustering algorithm</mark> that r<mark style="background: #FFF3A3A6;">eturns data points</mark>(<mark style="background: #FFF3A3A6;">k-medoids</mark>) as <mark style="background: #FFF3A3A6;">cluster centers,</mark> qualify, selecting prototypes.
- Can <mark style="background: #FFF3A3A6;">compute distance</mark>, from <mark style="background: #FFF3A3A6;">each point</mark>, to<mark style="background: #FFF3A3A6;"> nearest prototype</mark>, <mark style="background: #FFF3A3A6;">points with largest distances</mark>, <mark style="background: #FFF3A3A6;">as heuristic</mark>, <mark style="background: #FFF3A3A6;">works well where</mark> <mark style="background: #FFF3A3A6;">distance metric</mark> is <mark style="background: #FFF3A3A6;">meaningful</mark> for <mark style="background: #FFF3A3A6;">class representation</mark>. 


## Active Learning
> <mark style="background: #FFF3A3A6;">ML paradigm</mark>, <mark style="background: #FFF3A3A6;">model</mark>, actively <mark style="background: #FFF3A3A6;">selects</mark> which <mark style="background: #FFF3A3A6;">data points</mark>, <mark style="background: #FFF3A3A6;">should be labelled</mark> by human, <mark style="background: #FFF3A3A6;">instead of</mark> <mark style="background: #FFF3A3A6;">passively learning</mark>
### Motivation + Comparison
- <mark style="background: #FFF3A3A6;">Trad supervised learning</mark>, <mark style="background: #FFF3A3A6;">training examples</mark>, **<mark style="background: #FFF3A3A6;">randomly sampled</mark>** training set, <mark style="background: #FFF3A3A6;">fed into ML algorithm</mark>
- <mark style="background: #FFF3A3A6;">In active learning</mark>, ML <mark style="background: #FFF3A3A6;">algorithm</mark>, <mark style="background: #FFF3A3A6;">start from scratch</mark>, <mark style="background: #FFF3A3A6;">ask user</mark>, <mark style="background: #FFF3A3A6;">label specific points</mark>
- <mark style="background: #FFF3A3A6;">Fully labelling</mark> = <mark style="background: #FFF3A3A6;">time-consuming</mark>,<mark style="background: #FFF3A3A6;"> expert knowledge needed</mark>; <mark style="background: #FFF3A3A6;">just label informative instances</mark>,<mark style="background: #FFF3A3A6;"> insert</mark> those <mark style="background: #FFF3A3A6;">back into training</mark> set, retrain
- <mark style="background: #FFF3A3A6;">Active learning</mark>, can <mark style="background: #FFF3A3A6;">start</mark>, <mark style="background: #FFF3A3A6;">completely unlabelled data</mark>, while trad SL, needs bare labelled data.


### Active learning in practice
- <mark style="background: #FFF3A3A6;">General idea</mark>:
	- **<mark style="background: #FFF3A3A6;">Select</mark>** - <mark style="background: #FFF3A3A6;">batch</mark>, <mark style="background: #FFF3A3A6;">data points</mark>, via <mark style="background: #FFF3A3A6;">criteria</mark>
	- **<mark style="background: #FFF3A3A6;">Query</mark>** - <mark style="background: #FFF3A3A6;">human user</mark>, <mark style="background: #FFF3A3A6;">label that batch</mark>
	- **<mark style="background: #FFF3A3A6;">Retrain</mark>**

#### Query models
- Exist, **<mark style="background: #FFF3A3A6;">query models</mark>**, <mark style="background: #FFF3A3A6;">determine,</mark> how <mark style="background: #FFF3A3A6;">model selects</mark> <mark style="background: #FFF3A3A6;">data points</mark>
	- **<mark style="background: #FFF3A3A6;">Uncertainty sampling</mark>**
	- **<mark style="background: #FFF3A3A6;">Query by committee</mark>**
	- **<mark style="background: #FFF3A3A6;">Expected error reduction</mark>**
	- **<mark style="background: #FFF3A3A6;">Expected model change</mark>**
##### Uncertainty sampling
- <mark style="background: #FFF3A3A6;">Query instances</mark>, <mark style="background: #FFF3A3A6;">model</mark>, <mark style="background: #FFF3A3A6;">least confident about</mark>
- E.g., <mark style="background: #FFF3A3A6;">probabilistic classifier</mark>, find <mark style="background: #FFF3A3A6;">docs, probability, closest to random</mark> e.g., close to <mark style="background: #FFF3A3A6;">0.5</mark>
- E.g., <mark style="background: #FFF3A3A6;">NN classifiers</mark>, find <mark style="background: #FFF3A3A6;">docs</mark> <mark style="background: #FFF3A3A6;">equally close</mark> to <mark style="background: #FFF3A3A6;">multiple classes</mark>
- E.g., <mark style="background: #FFF3A3A6;">Multiclass classifiers</mark>, find <mark style="background: #FFF3A3A6;">doc</mark> with <mark style="background: #FFF3A3A6;">smallest decision boundary</mark> <mark style="background: #FFF3A3A6;">between top classes</mark>
- Often <mark style="background: #FFF3A3A6;">near decision boundary</mark>
- <mark style="background: #FFF3A3A6;">Queried instances</mark> = *<mark style="background: #FFF3A3A6;">unlabelled</mark>* <mark style="background: #FFF3A3A6;">data points</mark> for which <mark style="background: #FFF3A3A6;">model</mark> has <mark style="background: #FFF3A3A6;">made</mark> <mark style="background: #FFF3A3A6;">tentative predictions</mark> with <mark style="background: #FFF3A3A6;">high uncertainty</mark>, <mark style="background: #FFF3A3A6;">oracle provides true</mark>, <mark style="background: #FFF3A3A6;">refine decision boundary;</mark> <mark style="background: #FFF3A3A6;">optimise local uncertainity.
</mark>
##### Query by committee
- <mark style="background: #FFF3A3A6;">Train</mark>, <mark style="background: #FFF3A3A6;">multiple models</mark>, in <mark style="background: #FFF3A3A6;">parallel,</mark> on <mark style="background: #FFF3A3A6;">provisionally labelled dat</mark>a
- <mark style="background: #FFF3A3A6;">Each model predicts</mark> <mark style="background: #FFF3A3A6;">label,</mark><mark style="background: #FFF3A3A6;"> measure </mark>**<mark style="background: #FFF3A3A6;">vote entropy</mark>**, <mark style="background: #FFF3A3A6;">query instances</mark>, <mark style="background: #FFF3A3A6;">highest disagreement</mark>
- Disagreement indicate uncertainty/lack of knowledge to refine

##### Expected error reduction 
- <mark style="background: #FFF3A3A6;">Select instances</mark> <mark style="background: #FFF3A3A6;">whose</mark> labels would <mark style="background: #FFF3A3A6;">most reduce</mark> **<mark style="background: #FFF3A3A6;">future generalisation error.</mark>**
- <mark style="background: #FFF3A3A6;">Requires estimating</mark>, <mark style="background: #FFF3A3A6;">how</mark> <mark style="background: #FFF3A3A6;">model</mark> would <mark style="background: #FFF3A3A6;">perform</mark>, if <mark style="background: #FFF3A3A6;">each unlabelled point</mark>, were <mark style="background: #FFF3A3A6;">labelled</mark>
- <mark style="background: #FFF3A3A6;">Most principled</mark> <mark style="background: #FFF3A3A6;">active learning strategy</mark>, <mark style="background: #FFF3A3A6;">optimising</mark> **<mark style="background: #FFF3A3A6;">test performance</mark>**
- <mark style="background: #FFF3A3A6;">Comp infeasible</mark><mark style="background: #FFF3A3A6;"> large datasets</mark>, can <mark style="background: #FFF3A3A6;">approximate</mark>.

##### Expected model change
- <mark style="background: #FFF3A3A6;">Select instances for querying</mark>, <mark style="background: #FFF3A3A6;">where knowing label,</mark> would <mark style="background: #FFF3A3A6;">cause, largest update</mark>, <mark style="background: #FFF3A3A6;">model params</mark>, <mark style="background: #FFF3A3A6;">cheaper than error reduction</mark>, "<mark style="background: #FFF3A3A6;">asks</mark> <mark style="background: #FFF3A3A6;">which label</mark> would <mark style="background: #FFF3A3A6;">teach</mark> <mark style="background: #FFF3A3A6;">the most"</mark>?