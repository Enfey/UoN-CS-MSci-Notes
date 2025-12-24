# Conversational AI architectures
A typical pipeline includes:
1. **Preprocessing** → tokenisation, normalisation
2. **Representation** → e.g., Bag of Words
3. **Natural Language Understanding (NLU)**
    - Identifies intents, extracts meaning
4. **Dialogue Management**
    - Handles state tracking, memory, external system calls
5. **Natural Language Generation (NLG)**
    - Produces the final system response
A useful agent must:
- track **state** (conversation progress, user info),
- potentially use **memory**,
- interact with **external systems,**
- **learn/improve** over time.

## Architectures for Conversational Agents

### Rule-Based Systems
- Traditional paradigm, uses pattern matching and hand written rules
- Dialogue manager behaves as a state machine
- Pros: Predictable, Reliable, Interpretable
- Cons: Not robust, Not scalable
### Retrieval-Based systems
- Instead of generating text, select best possible response from an existing known corpus
	- Key technique here is searching
- Types of retrieval based systems include 
	- Simple matching e.g., BOW
	- Neural retrieval modes
- Pros: Response quality, Grounding(comes from real database)
- Cons: No novelty and not very flexible, Lack of context sensitivity

### Generative Models
- Use neural language models to produce responses
- Has evolved significantly over time
	- RNNs, LSTMs
	- Transformers/LLMs
- Pros: Flexible, good contextual awareness
- Cons: Not grounded(may hallucinate), can be inconsistent, require significant data
### Hybrid Systems - Retrieval Augmented Generation
- Combination of retrieval and generative models, aiming to provide grounding to generative models
	![[Pasted image 20251119212133.png]]
- Pros: Contextual, Grounded, Flexible
- Cons: Still inconsistent, Data intensive, Hard to implement correctly.
## Hybrid Systems - Cognitive Architectures
- More advanced agent frameworks capable of reasoning, planning, memory, tool use, and interacting with systems.
	![[Pasted image 20251119212315.png]]
- Pros: Flexible, Context-aware, Grounded, Interacts with a system
- Cons: Data intensive, Extremely hard to implement, Consistency issues remain
# Human in the Loop AI systems
- Traditional ML focuses on full automation, but this has limits:
	- Models saturate on benchmark datasets
	- Specialise too much
	- Suffer from "**concept drift**":
		- The statistical properties of the target variable change over time in unforeseen ways.
	- Cannot adapt efficiently due to overspecialisation
	- Often unsafe without oversight
- Human in the loop means that humans participate in:
	- Training
	- Tuning
	- Labelling
	- Correcting
	- Evaluating
	- Interpreting
- The notion is that by keeping humans in the process, move beyond the limitations of full automation

## Interpretability
- Interpretability of model and results builds **trust** (built via how mistakes are handled, not via success) and enables:
	- sanity-checking results (e.g., human expert checks e.g., doctor),
	- accountability,
	- debugging the inference process, 
	- understanding model errors
- However, what does interpretable mean?
- Interpretable by whom?
- And at what cost (performance, complexity, time)?
### Taxonomy of Interpretability
#### Intrinsic vs Post-Hoc
- **Intrinsic**: interpretability is a property of the model
- **Post-Hoc**: we interpret a prediction after the fact
#### Model-Specific vs Model-Agnostic
- **Model-Specific**: interpretation methods tied to particular models
- **Model-Agnostic**: interpretation methods applicable to any model
#### Local vs Global
- **Local**: interpret one prediction at a time.
- **Global**: interpret the entire model

### Examples of Intrinsically Interpretable models
- **Decision Trees**
	- Model classification as a finite sequence of discrete decisions
	- Easy to interpret until tree reaches explosive depth
	- Learned through algorithms that seep to minimise information entropy
	 ![[Pasted image 20251119225506.png]]
- K-Nearest-Neighbours
	- Classifies based on distances
	- Interpretability is trivial unless the space is very high dimensional
	![[Pasted image 20251119225522.png]]


### Model-Agnostic methods
- Interpretation methods applicable to any model; separation of explanation from the model, and can take many forms e.g., plot, text, feature weights etc.
- Desirable properties of a model-agnostic explanation system include:
	- **Model flexibility**
		Can work with any model
	- **Explanation flexibility**
		Can provide multiple types of explanation
	- **Representation flexibility**
		- Can map the explanation to differing types of explanations
#### Global Surrogate Models
- Train an interpretable model e.g., decision tree to approximate the predictions of a **black box model**.
- E.g., Train SVM on $(X,Y)$
- Classify a dataset, giving $(X, \hat{Y}$)
- Train decision tree on $(X, \hat{Y}$)
- If the accuracy of the decision tree is acceptable, use it to explain the decisions of the SVM.
- Goal of the surrogate model is to approximate the entire black box model, rather than specific predictions. 
#### Local Surrogate Models
- Used to explain individual predictions of black box machine learning models
	- Why did the model make this specific prediction?
	- What effect did this specific feature value have on the prediction?
- We use a simple interpretable model that is trained to approximate a black-box model in a small neighbourhood around a specific prediction. 
- Best known is **LIME** - "Locally Interpretable Model Estimation"
	- Generate perturbed versions of one example as synthetic data points, representing 'what could have happened'.
	- For each synthetic point, feed it into the black-box model, and record the predictions. 
	- Weight samples by proximity; distant points may follow different decision logic.
	- Train a linear, interpretable model on the weighted, perturbed dataset. 
	- Can then determine:
		- Which features contributed most to the prediction
		- Whether they pushed the prediction toward or away from a class
		- How sensitive the prediction is to feature changes
- This is a **feature-based explanation**, decomposing predictions into feature effects and not treating vectors as indivisible objects. 

#### Explaining with examples
- Model agnostic techniques; interpret black-box model by presenting **concrete instances** rather than model parameters, feature weights or rules. 
- Generally aims to provide a more intuitive, holistic focus when delivering model explanations. 
- Examples can be used to show
	- Implicit decision boundary (counterfactual explanation, adversarial examples)
	- Representative instances and unrepresented examples(prototypes/criticisms)
	- Which instances have the highest influence on model parameters
##### Counterfactual explanation
- Describes the closest alternative instance the to the original input that would result in a **different prediction**. 
- Explains invidiual predictions, and is used to identify potential causal links between concrete instances.
- "If feature x was higher by y, the prediction would have been positive"
	- Clearly interpretable
	- Very flexible
	- Easy to implement - often slightly perturb, until prediction changes, and select the closest valid alternative to propagate counterfactually.
	- Though often multiple counterfactual explanations for a single outcome.
##### Prototypes
- **Prototypes** are **representative instances** of a class that capture what the model (or data) considers *typical*.
- A **criticism** is an instance **not well represented** by the prototypes. 
- Prototypes can be selected manually, but this does not scale well and leads to poor results; there are many approaches to finding prototypes in the data, one of these is k-medoids, a clustering algorithm related, but any clustering algorithm that returns actual data points as cluster centers would qualify for selecting prototypes.
- ![](Pasted%20image%2020251224003623.png)

- One may choose to compute distance from each point to its nearest prototype, and pick points with largest distances as a heuristic, works well where the distance metric is meaningful for class representation and the data is well-scaled. 


### Active Learning
> Machine learning paradigm in which the model actively selects which data points should be labelled by a human (termed the oracle/teacher), instead of passively learning from a randomly labelled dataset
#### Motivation and Comparison
- In traditional supervised learning, training examples are **randomly sampled** from the training set and fed into the the machine learning algorithm, learning passively.
- In active learning, the machine learning algorithm starts from scratch, and asks the user to label specific data points. 
- Fully labelling data is often time-consuming, expensive, and requires expert knowledge, active learning can be achieved by humans just labelling informative instances, inserting those back into the training set and then retraining. 
- Active learning can start from completely unlabelled data, while traditional supervised learning needs a lot of labelled data from that start. 

#### Active Learning in practice
- General idea:
	- **Select** a new batch of data points based on some criteria
	- **Query** human user to label that batch
	- **Retrain** the model
- Focuses human effort where it matters most.

#### Query models
There exist different **query models** determining how the model selects data points:
- **Uncertainty sampling**
- **Query by committee**
- **Expected error reduction**
- **Expected model change**

##### Uncertainty Sampling
- Query instances the model is **least confident** about.
- For example, if using a probabilistic classifier, find documents where the probability is closest to random e.g., close to 0.5 binary classification
- For example, for nearest-neighbour classifiers, find documents/points which are equally close to multiple classes
- For example, multiclass classifiers, find document with the smallest margin between top classes. 
- Uncertain points are often near the decision boundary. 
- Thus, in ununcertainty sampling, the queried instances are _unlabelled_ data points for which the model has made _tentative predictions_ with high uncertainty; the oracle provides true labels, used to refine the decision boundary.
- Optimises local uncertainity
##### Query by committee
- Train multiple models (the committee) in parallel on the current provisionally labelled data.
- Each model predicts labels for unlabelled data, measure disagreement/vote entropy, and query instances with the highest disagreement
- Disagreement indicates model uncertainty or lack of knowledge to self-refine.
##### Expected error reduction
Select instances for querying whose labels would **most reduce future generalisation error**. This requires estimating how the model would perform if each unlabelled point were labelled. Most principled active learning strategy, directly optimising **test performance**. Computationally nfeasible for large datasets, but can approxmiate.
##### Expected model change
Select instances for querying where knowing the label would case the **largest** update to model parameters. Cheaper than error reduction, asks "which label would teach me the most?".


