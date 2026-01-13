Core, all lang models (RNNs, Transformers) simple task:
> Predict next tok, given prev toks.

Formally:
$$P(w_{t+1} ∣ w_{1}​,w_{2}​,\dots, w_{t})$$
Construct everything atop this.

### Training RNN/Lang model
- Train RNN, gen next lang model
	- Train via CE loss of softmaxed/normalised probability for correct word, where correct word is modelled as one-hot.
- Allows, large scale, self-supervised learning
	- Pretext tasks, masking, nwp, predict parts of input, label come from data itself. 
	- Model fine tuned, downstream tasks = foundation model

### Training LLM
- Pretrained, unsupervised manner
	- Huge corpora used
- During pretraining, learn grammar, syntax, word meanings, facts, patterns, reasoning structure, how lang flows, via pretext tasks. 

#### Training phases of LLM
#### Unsupervised pre-training
- Model fed text, trained, ntp(next tok predict). **Teach lang struct, grammar, word associations.** 
	- Take long time
- Same loss function: CE-loss
	- Want model, assign high prob, true word $w$, want loss high, if model, assigns low prob.
	- CE loss = negative log prob, that model assigns, true word $w$ 
	- Move weights to optimise via gradient descent or similar after backprop
- **Teacher-forcing**
	- Receive true prev word for future probability computations, early mistakes don't compound, stable training, ignore prior predictions. 

#### Fine-tuning
- Supervised fine-tuning
	- Pretrained model = refined for spec tasks:
		- Summarisation
		- QA
		- Translation
		- Reasoning
- Fine-tuning, help model learn nuances, specific reqs, of task.
	- Fine tuning on same task, but on more specialised dataset (continued pretraining)
	- Fine tuning, different downstream task, closer to real-world usage.

#### Reinforcement learning with Human Feedback
1. **Reward model Training**
	1. **<mark style="background: #FFF3A3A6;">Human feedback</mark>** - humans, rate, compare, diff responses, by lang model
	2. **<mark style="background: #FFF3A3A6;">Reward model</mark>**- separate model, trained, predict human preferences, based on feedback data; assigns reward scores to piece of text.
2. **Policy optimisation via RL**
	1. **<mark style="background: #FFF3A3A6;">Fine-tuning LLM</mark>** - OG lang model, fine tuned, but uses reward scores, from trained reward model.
	2. **<mark style="background: #FFF3A3A6;">Reinforcement learning loop</mark>** - updated model, generates new text, reward model again provide feedback, guide weight adjustment, better output. 

#### Decoding: How model generates text
- Task of choosing word to generate based on probabilities = **<mark style="background: #FFF3A3A6;">decoding</mark>**
- Model outputs probability distribution over tokens at each step
- Most common method for decoding in LLMs = **sampling**, sample distribution over words according to assigned probabilities


##### Greedy decoding
> Pick most probable word every time

Leads to boring, repetitive text. 

##### Top-K sampling
1. Choose # of words $K$ 
2. For each $w \in V$, use lang model predict probability  $P(w_t \ | \ w_{<t})$ 
3. Sort words by likelihood, keep only top $k$ most probable words
4. Normalise scores of the $k$ words, form probability distribution
5. Randomly sample word within these remaining $k$ most probable words
Fixed $k$ can be restrictive or too broad depending


##### Top-p sampling
- Instead of fixed num of words
	- Select smallest set of words whose probability $\ge$ $p$ 
- Keep top $p$ percent of probability mass
- Given distribution, $P(w_t \ | \ w_{<t})$ , the top-$p$ vocabulary $V^{p}$ is the smallest set of words such that:
	$$\sum_{w \in V^{(P)}}P(w_{t} \ | \ w_{<t})\ge p$$
##### Temperature sampling
- Intuition, thermodynamics
	- System at high temp, flexible, explore many configs
	- Low temp, explore subset, low energy states
- Applies scaling factor $\tau$ to model's raw output scores before squashed into probability distribution via softmax:
	$$P_{i} = \frac{e^{\frac{z_{i}}{\tau}}}{\sum_{j}e^{\frac{z_{i}}{\tau}}}$$
	Divide logits (weighted input + vector) by scaling factor, to compute probability distribution
- **Low temperature sampling** $\tau < 1$ 
	- Sharpen probability distribution, increase likelihood, high probability words, more predictable, factual, coherent
- **High temperature sampling** $\tau > 1$ 
	- Flatten distribution, give low probability words, better chance
- $\tau = 1$ uses original probabilities.

### Challenges in LLMs
- <mark style="background: #FFF3A3A6;">Hallucination</mark>
	- <mark style="background: #FFF3A3A6;">Happens</mark> as <mark style="background: #FFF3A3A6;">model</mark> <mark style="background: #FFF3A3A6;">must generate something</mark>;<mark style="background: #FFF3A3A6;"> sampling from probabilities</mark>, <mark style="background: #FFF3A3A6;">no grounding, </mark>may be <mark style="background: #FFF3A3A6;">issues in training data.</mark>
- Few ways to alleviate:
	- **<mark style="background: #FFF3A3A6;">In-context learning</mark>**
		- <mark style="background: #FFF3A3A6;">Teach LLM at inference time</mark>, task demonstrations, integrated into prompt in nat lang format, allow pre-trained llms address new tasks, without fine tuning, but <mark style="background: #FFF3A3A6;">accumulated knowledge is transient</mark>, merely biasing probability distribution. 
	- **<mark style="background: #FFF3A3A6;">Reasoning</mark>**
		- Force LLM, think, <mark style="background: #FFF3A3A6;">structured way</mark>, generate intermediate steps before answer, <mark style="background: #FFF3A3A6;">problem decomposed</mark>.
	- **<mark style="background: #FFF3A3A6;">Grounding</mark>**
		- E.g., RAG
 - LLM output, determined, training data, and input, **prompt <mark style="background: #FFF3A3A6;">engineering</mark>**, tailor input
	 - **<mark style="background: #FFF3A3A6;">Chain-of-though prompting</mark>**
		 - Ask model, show reasoning step, before giving final answer.
	 - **<mark style="background: #FFF3A3A6;">Directional stimulus prompting</mark>**
		 - Guide model toward reasoning strat without explicit steps e.g., "think about edge cases"
		 - <mark style="background: #FFF3A3A6;">Nudge models toward internal reasoning patterns</mark>
	 - **Meta prompting**
		 - <mark style="background: #FFF3A3A6;">Prompt model</mark>, about <mark style="background: #FFF3A3A6;">how should think or behave</mark> e.g., you are expert mathematician
	 - **Tree of thoughts**
		 - <mark style="background: #FFF3A3A6;">Instead of single reasoning chain</mark>, ask <mark style="background: #FFF3A3A6;">explore multiple,</mark> evaluate them
			 1. Gen possible states
			 2. Evaluate each step
			 3. Expand best ones
			 4. Prune weak paths
	- **<mark style="background: #FFF3A3A6;">Self-consistency prompting</mark>**
		- Model gathers <mark style="background: #FFF3A3A6;">multiple independent reasoning chains</mark>, <mark style="background: #FFF3A3A6;">picks most common final answer</mark> e.g., sample 5 chain of thought answers, extract final answer, choose most frequent. 
### RAG
- Ground, give external knowledge at inference time, retrieve and artificially inject relevant facts into prompt itself. 
- Pipeline:
	1. User ask query, or LLM gen query from proompt
	2. Convert question, embedding
	3. Search vector database
	4. Retrieve top-$k$ relevant chunks
	5. Inject into prompt
	6. Gen answer

#### Graph RAG
- Standard RAG, treats knowledge, **chunks of text**
- Many domains = relational, need finer granularity, need to be able to reason/join concepts
- **<mark style="background: #FFF3A3A6;">Knowledge graphs</mark>** used, queried by LLMs, traverse graph instead
- Complex relationships easily captured
### Recent trnds in LLMs
#### Multimodal LLMs
- LLMs, that understand multiple data types, like text, images, audio, video, transcend text-only
<mark style="background: #FFF3A3A6;">- Image encoder,audio encoder etc,. align onto shared embedding space, same one as text tokens</mark>

### Quantisation
- <mark style="background: #FFF3A3A6;">Shrink LLMs, reduce precision of weights/activations, cut memory usage, speed up inference
- LLMs = huge
- LoRA/QLoRA, find smaller model that performs similarly.</mark>


### Specialised LLMs
- Construct 
	- Medical LLMs
	- Legal LLMs
- Reliable in-domain

### Agentic architectures
- <mark style="background: #FFF3A3A6;">Agentic LLM, LLM composed, modular layers, mimic cognitive proceesses, mark itself, autonomous system, capable of reasoning planning, iterating</mark>

#### Tool calling
<mark style="background: #FFF3A3A6;">- Core mechanism, permit LLM, interact, externally, invoke APIs,</mark> databases, custom code, permit capability of real-world tasks.![](Pasted%20image%2020251227024147.png)
- User message, tool needed, no, text rsponse
- User message, tool needed, yes, generate JSON schema for tool, execute function, send result back

### Mixture of experts models
- Instead of dense model, have sparsely connected 'expert networks' with identical arch
<mark style="background: #FFF3A3A6;">- Gating mechanism, routes input to experts(weighted sum, top-$k$ experts)</mark>
	-<mark style="background: #FFF3A3A6;"> Token embedding, goes into router</mark>
	- Have $n$ experts, router outputs scores
	- Choose top $k$ experts with highest scores
	- Compute weighted sum of expert outputs
		- Each exprrt produce output vector $y_i = Expert_i(x)$ 
		- Final output is those weighted for the top $k$.
- Small subset, faster inference, lower compute cost.

## Evaluating + benchmarking LLMs
- Like each field, LLMs have own way, measuring progress
- Advantages:
	- Track progress, improve
	- Identify strengths/weaknesses (select the correct model)
- Drawbacks
	- Most evals oversimplify
	- Models can overfit benchmarks
	- Not ecologically valid

### Evaluation dimensions
- **<mark style="background: #FFF3A3A6;">Accuracy and fluency</mark>**
- **<mark style="background: #FFF3A3A6;">Reasoning</mark>**
- **<mark style="background: #FFF3A3A6;">Knowledge and common sense</mark>**
- **<mark style="background: #FFF3A3A6;">Bias and fairness</mark>**
- **<mark style="background: #FFF3A3A6;">Efficiency and scalability</mark>**