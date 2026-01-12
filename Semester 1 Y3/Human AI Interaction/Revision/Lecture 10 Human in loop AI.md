# Conversational AI architectures
- Typical pipeline:
	1. **Preprocessing**
		Tokenisation, normalisation
	2. **Representation**
		BOW
	3. **NLU**
		Intent, extract meaning
	4. Dialogue management
		State, entities, system calls
	5. **NLG**
		Produce final response
Useful agent must:
- Track **state**
- Use **memory**
- Interact with **external systems**
- **Learn/improve** over time

## Architectures for Conversational Agents
### Rule-Based Systems
- Trad paradigm, pattern matching, hand-written rules
- Dialogue manager = state machine
- Predictable, reliable interpretable
- Not robust, not scalable
### Retrieval-based systems
- Instead of gen text, select response, existing corpus
- Types of retrieval based systems include
	- Simple matching e.g., BOW
	- Neural retrieval modes
- Response quality good, grounding
- No novelty, not flexible, lack context sensitivity w/o pre-processing

### Generative models
- Use neural language models, produce response
- Evolved over time
	- RNNs, LSTMs
	- Transformers
- Flexible, context awareness
- Cons: Not grounded, inconsistent, need signif data.

### Hybrid Systems - Retrieval Augmented Generation
- Combination, retrieval-based, generative models, provide grounding, generative models. 
	![[Pasted image 20251119212133.png]]
	USE QUERY TO GET ENHANCED CONTEXT, ATTACH TO OG INPUT, THEN RETURN RESPONSE.
- Contextual, grounded, flexible
- Inconsistent, data intensive, hard implement

### Hybrid Systems - Cognitive Architectures
- Advanced, 'agent framework', reason, plan, memory, tool use, interact w/systems
	![[Pasted image 20251119212315.png]]
- Flexible, context-aware, grounded, interact with system
- Data intensive, hard implement, consistency

# Human in the Loop AI Systems
- Trad ML, focus, full automation, has limits:
	- Models, saturate, benchmark datasets
	- Overspecialise
	- Suffer from "**concept drift**"
		- Statistical properties of target var change over time in unforeseen ways
	- Cannot adapt efficiently due to overspecialisation
	- Unsafe without oversight
- Human in loop; participate in:
	- Training
	- Tuning
	- Labelling
	- Correcting
	- Evaluating
	- Interpreting
## Interpretability
- Interpretability of model + results, build **trust** (via how mistakes handled, not success), enables:
	- <mark style="background: #FFF3A3A6;">Sanity-check</mark> results
	- <mark style="background: #FFF3A3A6;"> Accountability</mark>
	- <mark style="background: #FFF3A3A6;">Debugging </mark>inference process
	- <mark style="background: #FFF3A3A6;">Understanding model errors</mark>
- However, what does interpretable mean?
- Interpretable by whom?
- And, at what cost?
### Taxonomy of interpretability
#### Intrinsic vs Post-Hoc
- **Intrinsic** - interpretability = property of model
- **Post-hoc** - interpret predictions after the fact
#### Model-specific vs Model-agnostic
- **Model-specific** - interpretation methods tied to particular models
- **Model-agnostic** - interpretation methods applicable to any model
#### Local vs Global
- **Local** - interpret one prediction at a time
- **Global** - interpret the entire model

### Examples of Intrinsically Interpretable Models
- **Decision trees**
	- Model classification, finite sequence, discrete decisions
	- Interpret easily until tree, reach, explosive depth
	- Learned through algos seek minimise information entropy
- **KNN**
	- Classifies based on distances
	- Interpretability trivial in low-dimensional spaces.


### Model-Agnostic Methods
- Interpretation methods, **applicable, any model**; separation of explanation from model
- Desirable properties, model-agnostic explanation system
	- **<mark style="background: #FFF3A3A6;">Model flexibility</mark>**
		- Work w/ any model
	- **<mark style="background: #FFF3A3A6;">Explanation flexibility</mark>**
		- Multiple types of explanation
	- **<mark style="background: #FFF3A3A6;">Representation flexibility</mark>**
		- Map explanation, differing types, representations
#### Global Surrogate models
- Train, interpretable model e.g., decision tree, approximate, decisions, **black box model**
- E.g., train SVM on $(X,Y)$
- Classify dataset, giving $(X, \hat{Y}$)
- Train decision tree on  $(X, \hat{Y})$
- If accuracy of decision tree acceptable, use to explain decisions of SVM
- Approximate entire black box.
#### Local Surrogate models
- Used, explain, individual predictions, black box machine learning models. 
	- Why did model, make, specific prediction?
	- What effect, did, feature val, have on prediction?
- Use simple, interpretable model, trained, approximate black-box model, in small neighbourhood around specific prediction. 
- Best known = **LIME - "Locally Interpretable Model Estimation"**
	- Generate, perturbed versions, one example; synthetic data points
	- For each synthetic point, feed back into black box, record predictions
	- Weight samples via proximity; distant points may follow different decision logic
	- Train linear, interpretable model, on weighted, perturbed dataset
	- Determine:
		- Which features contributed most to prediction
		- Whether they pushed prediction, toward/away class
		- How sensitive, prediction is, feature changes.
	- **Feature-based explanation**, decomposing predictions into feature effects; vectors not indivisible.


#### Explaining with examples
- **Model agnostic technique**; interpret bb model, presenting, concrete instances, rather than, model params, feature weights, or rules. 
- Provide more intuitive, holistic focus
- Examples show:
	- Implicit decision boundary(adversarial examples, counterfactual explanation)
	- Representative instances(**prototypes**), and unrepresented examples(**criticisms**)
	- Which **instances**, have highest influence, model parameters.

#### Counterfactual explanation
- Describe, closest alternative instance, original input , that would result, **different prediction**
- Explains, individual predictions, used, identify, causal links between concrete instances
- "If feature x was higher by y, prediction would have been positive"
	- Interpretable
	- Flexible
	- Easy to implement, slightly perturb, until prediction changes, select closest valid alternative to propagate counterfactually
	- Often multiple counterfactual explanations, for single outcome. 
#### Prototypes
- **Prototypes** = **representative instances** of class, capture, what model/data, considers *typical*
- **Criticism** = instance, **not well represented**, by prototypes.
- Prototypes, can be selected manually, does not scale well, poor results; many approaches to finding, any clustering algorithm that returns data points(k-medoids) as cluster centers, qualify, selecting prototypes.
- Can compute distance, from each point, to nearest prototype, points with largest distances, as heuristic, works well where distance metric is meaningful for class representation. 


## Active Learning
> ML paradigm, model, actively selects which data points, should be labelled by human, instead of passively learning
### Motivation + Comparison
- Trad supervised learning, training examples, **randomly sampled** training set, fed into ML algorithm
- In active learning, ML algorithm, start from scratch, ask user, label specific points
- Fully labelling = time-consuming, expert knowledge needed; just label informative instances, insert those back into training set, retrain
- Active learning, can start, completely unlabelled data, while trad SL, needs bare labelled data.


### Active learning in practice
- General idea:
	- **Select** - batch, data points, via criteria
	- **Query** - human user, label that batch
	- **Retrain**

#### Query models
- Exist, **query models**, determine, how model selects data points
	- **Uncertainty sampling**
	- **Query by committee**
	- **Expected error reduction**
	- **Expected model change**
##### Uncertainty sampling
- Query instances, model, least confident about
- E.g., probabilistic classifier, find docs, probability, closest to random e.g., close to 0.5
- E.g., NN classifiers, find docs equally close to multiple classes
- E.g., Multiclass classifiers, find doc with smallest decision boundary between top classes
- Often near decision boundary
- Queried instances = *unlabelled* data points for which model has made tentative predictions with high uncertainty, oracle provides true, refine decision boundary; optimise local uncertainity.

##### Query by committee
- Train, multiple models, in parallel, on provisionally labelled data
- Each model predicts label, measure **vote entropy**, query instances, highest disagreement
- Disagreement indicate uncertainty/lack of knowledge to refine

##### Expected error reduction 
- Select instances whose labels would most reduce **future generalisation error.**
- Requires estimating, how model would perform, if each unlabelled point, were labelled
- Most principled active learning strategy, optimising **test performance**
- Comp infeasible large datasets, can approximate.

##### Expected model change
- Select instances for querying, where knowing label, would cause, largest update, model params, cheaper than error reduction, "asks which label would teach the most"?