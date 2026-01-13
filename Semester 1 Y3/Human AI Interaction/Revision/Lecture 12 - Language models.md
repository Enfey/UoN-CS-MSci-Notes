## Statistical lang models
<mark style="background: #FFF3A3A6;">Use n-gram prob models</mark>, <mark style="background: #FFF3A3A6;">before neural </mark>- <mark style="background: #FFF3A3A6;">given sequence</mark> of <mark style="background: #FFF3A3A6;">toks</mark>, <mark style="background: #FFF3A3A6;">predict next tok</mark>. <mark style="background: #FFF3A3A6;">Assign probability</mark> <mark style="background: #FFF3A3A6;">sequence of words</mark>, <mark style="background: #FFF3A3A6;">decide, what sound nat</mark>, in lang. 


<mark style="background: #FFF3A3A6;">Probability</mark> of <mark style="background: #FFF3A3A6;">whole sentence:</mark>
$P(w_1, w_2, w_3,...w_n) = P(w_1) \times P(w_2∣w_1) \times P(w_3∣w_1, w_2) \times \dots \times P$ 

<mark style="background: #FFF3A3A6;">Impossible to compute directly,</mark> aka:
$$\prod_iP(w_i ∣w_1, w_2 \dots w_{i-1})$$
Simplify; <mark style="background: #FFF3A3A6;">each word</mark>, <mark style="background: #FFF3A3A6;">depend</mark>, only on <mark style="background: #FFF3A3A6;">last few</mark>, not entire sentence.
$P(w_i∣w_{i-n+1}, \dots w_{i-1})$ 0 = no context, 1 = bigram, defines range of context
<mark style="background: #FFF3A3A6;">Each word </mark>$i$, <mark style="background: #FFF3A3A6;">depends</mark> on<mark style="background: #FFF3A3A6;"> restricted set,</mark> <mark style="background: #FFF3A3A6;">preceding words.</mark>
**Assumption**: <mark style="background: #FFF3A3A6;">language </mark>is <mark style="background: #FFF3A3A6;">stochastic</mark>, <mark style="background: #FFF3A3A6;">memoryless </mark>process. 
We <mark style="background: #FFF3A3A6;">use</mark> the <mark style="background: #FFF3A3A6;">above 2</mark> statements to <mark style="background: #FFF3A3A6;">make</mark> <mark style="background: #FFF3A3A6;">language</mark> <mark style="background: #FFF3A3A6;">countable</mark>.
Example with $n$ = 0
	$𝑃(S_{1}) = 𝑃(𝑁𝑎𝑡𝑢𝑟𝑒) × 𝑃(𝑖𝑠) × 𝑃(ℎ𝑒𝑎𝑙𝑖𝑛𝑔)$
Example with n = 1
	$P(S_1) = 𝑃 (𝑁𝑎𝑡𝑢𝑟𝑒) × 𝑃 (𝑖𝑠 \ | \ 𝑁𝑎𝑡𝑢𝑟𝑒)× P( ℎ𝑒𝑎𝑙𝑖𝑛𝑔 \ | \ 𝑖𝑠)$ 
Counting these probabilities, feasible.

$P(S_1) = P(Nature) \times \frac{\#(Nature \ is)}{\#(Nature)} \times \frac {\#(is \ healing)} {\#(is)}$

<mark style="background: #FFF3A3A6;">WHEN COMPUTING PROB</mark>, <mark style="background: #FFF3A3A6;">TAKE</mark> ACTUAL <mark style="background: #FFF3A3A6;">BIGRAM</mark> IN SENTENCE, <mark style="background: #FFF3A3A6;">LOOK UP COUNT</mark>, AND <mark style="background: #FFF3A3A6;">DIVIDE BY COUNT</mark> OF <mark style="background: #FFF3A3A6;">CONTEXT WINDOW</mark>.
	$P(w_i | w_{i-1}) = \frac{w_{i-1}, w_i}{w_{i-1}}$

![](Pasted%20image%2020251224022621.png)
Bigram count
![](Pasted%20image%2020251224022715.png)
Unigram count
![](Pasted%20image%2020251224023042.png)
![](Pasted%20image%2020251224023108.png)
If<mark style="background: #FFF3A3A6;"> bigram</mark> has <mark style="background: #FFF3A3A6;">zero probability</mark>, <mark style="background: #FFF3A3A6;">P(sentence) = 0,</mark> combination never appeared before, <mark style="background: #FFF3A3A6;">ML alone insufficient.</mark>


### Laplace smoothing
- <mark style="background: #FFF3A3A6;">Addivite smoothing</mark>
- When <mark style="background: #FFF3A3A6;">counting occurrences</mark>, <mark style="background: #FFF3A3A6;">add quantit</mark>y to<mark style="background: #FFF3A3A6;"> numerator</mark>
- $P(w_i | w_{i-1}) = \frac{\#(w_{i-1}, w_i)+1}{\#(w_{i-1})}$
- <mark style="background: #FFF3A3A6;">Probabilities</mark> need to <mark style="background: #FFF3A3A6;">sum up to 1</mark>, if vocab size = V, then <mark style="background: #FFF3A3A6;">added 1</mark>, "<mark style="background: #FFF3A3A6;">V times</mark>"
- $$P(w_i | w_{i-1}) = \frac{\#(w_{i-1}, w_i)+1}{\#(w_{i-1})+V}$$
- <mark style="background: #FFF3A3A6;">Guarantees non-zero probabilities</mark> and easy to implement, but <mark style="background: #FFF3A3A6;">overpenalises frequent events. </mark>

## Neural language models
- <mark style="background: #FFF3A3A6;">Predict</mark>, <mark style="background: #FFF3A3A6;">lang,</mark> by <mark style="background: #FFF3A3A6;">assigning probabilities</mark>, to <mark style="background: #FFF3A3A6;">word sequences</mark>, based on <mark style="background: #FFF3A3A6;">computation</mark>, <mark style="background: #FFF3A3A6;">not counting.</mark>
- NN, <mark style="background: #FFF3A3A6;">learn word representation</mark>, <mark style="background: #FFF3A3A6;">predict next token directly</mark>. Explicit probability tables, replace w/ <mark style="background: #FFF3A3A6;">learned parameters.</mark>

- <mark style="background: #FFF3A3A6;">Statistical models</mark>, suffered,<mark style="background: #FFF3A3A6;"> sparsity</mark>; <mark style="background: #FFF3A3A6;">zero probabilities</mark> with parsing, need for <mark style="background: #FFF3A3A6;">smoothing</mark>,<mark style="background: #FFF3A3A6;"> poor generalisation. </mark>
- <mark style="background: #FFF3A3A6;">Neural LMs </mark>solve by <mark style="background: #FFF3A3A6;">embedding words,</mark> in vector space, allow similar words have similar reps, <mark style="background: #FFF3A3A6;">generalise</mark> to unseen combinations

**Embedding** = <mark style="background: #FFF3A3A6;">dense numerical vector</mark>, <mark style="background: #FFF3A3A6;">representing token.</mark> <mark style="background: #FFF3A3A6;">Embedding dimension</mark> $n$ determines $\mathbb{R}^n$

Words like "car", "bus", close in space, model can generalise. 
![](Pasted%20image%2020251224030531.png)
- <mark style="background: #FFF3A3A6;">Core issue</mark>; NN, require <mark style="background: #FFF3A3A6;">fixed-size input vectors</mark>. Nat lang, inherently variable in length. 
- <mark style="background: #FFF3A3A6;">One solution</mark>, add <mark style="background: #FFF3A3A6;">special tokens</mark>, <mark style="background: #FFF3A3A6;">short sequences</mark>, <mark style="background: #FFF3A3A6;">normalise input length</mark>, truncate long sequences.
- <mark style="background: #FFF3A3A6;">Loss of info</mark>. Another solution, <mark style="background: #FFF3A3A6;">combine all word embeddings</mark>, into one<mark style="background: #FFF3A3A6;"> fixed size vector</mark>; but<mark style="background: #FFF3A3A6;"> lose word order</mark>, sentences, same words, different meaning, equivalent under this scheme.
	![](Pasted%20image%2020251224030839.png)
- <mark style="background: #FFF3A3A6;">Natural solution:</mark><mark style="background: #FFF3A3A6;"> build architecture</mark>, naturally <mark style="background: #FFF3A3A6;">handles **sequences**</mark>
	- Examples:
		- RNNs
		- LSTMs
		- GRUs
		- Transformers

### Recurrent Neural Networks
<mark style="background: #FFF3A3A6;">NN </mark>designed, <mark style="background: #FFF3A3A6;">sequential data,</mark> maintain **<mark style="background: #FFF3A3A6;">recurrent connection</mark>**, let info persist across time.


Each timestep $t$:
- Input $x_t$ 
- Hidden state $h_t$ 
- Output $y_t$
Hidden state = **memory**


#### Input/output patterns
![](Pasted%20image%2020251224032532.png)

#### Elman vs Jordan networks
Early RNN arch.
![](Pasted%20image%2020251224032727.png)
Hidden state update = $h_t = \sigma(W_{x}x_t+U_{h}h_{t-1}+b_{h})$
	U = recurrent weight matrix.
Output computation = $y_t = (W_{y}h_{t}+b_{y})$
	W = output weight matrix
![](Pasted%20image%2020251224033008.png)
Hidden state update just comes from recurrent matrix weighted output now instead of drom hidden layer.


WE USE ELMAN!!!!!!!!!!!


#### Unfolding RNN through time
- <mark style="background: #FFF3A3A6;">RNN over sequence</mark>, length $T$ become depth-$T$ NN. <mark style="background: #FFF3A3A6;">Each layer</mark> = <mark style="background: #FFF3A3A6;">1 timstep. </mark>
- <mark style="background: #FFF3A3A6;">Same weights</mark>, used, <mark style="background: #FFF3A3A6;">every timestep</mark>; unroll into $T$ <mark style="background: #FFF3A3A6;">copies </mark>of <mark style="background: #FFF3A3A6;">same network, </mark>one for each time step, where in unfolded, view chain of $T$ identical RNN cells where each cell, receive $x_t$ at time $t$, $h_{t-1}$, and <mark style="background: #FFF3A3A6;">produces output</mark> $y_t$ (useful, <mark style="background: #FFF3A3A6;">showing loss</mark>, <mark style="background: #FFF3A3A6;">identifying cobtribution</mark>). Pass $h_t$ via equation. 

- <mark style="background: #FFF3A3A6;">Depending on task</mark>, <mark style="background: #FFF3A3A6;">output</mark>, <mark style="background: #FFF3A3A6;">matter differently:</mark>
	- <mark style="background: #FFF3A3A6;">Many to many</mark>(sequence-to-sequence): o<mark style="background: #FFF3A3A6;">utput at every timestep matters</mark>
	- <mark style="background: #FFF3A3A6;">Many to one:</mark> <mark style="background: #FFF3A3A6;">only final output</mark> matter
	- <mark style="background: #FFF3A3A6;">One to many</mark>: <mark style="background: #FFF3A3A6;">all output </mark>matter
- <mark style="background: #FFF3A3A6;">Unrolling</mark>, reveal, <mark style="background: #FFF3A3A6;">each hidden state</mark>, <mark style="background: #FFF3A3A6;">depend</mark> on <mark style="background: #FFF3A3A6;">prev inputs </mark><mark style="background: #FFF3A3A6;">earlier inputs,</mark> <mark style="background: #FFF3A3A6;">influence</mark> later outputs.
- Each timestep, <mark style="background: #FFF3A3A6;">same params.</mark>

#### Training RNNs: Backpropagation through time
- <mark style="background: #FFF3A3A6;">RNN</mark> = <mark style="background: #FFF3A3A6;">deep feedforward network</mark>, when <mark style="background: #FFF3A3A6;">unfolded</mark>
	1. <mark style="background: #FFF3A3A6;">Forward pass</mark>, <mark style="background: #FFF3A3A6;">all timesteps</mark>
	2. <mark style="background: #FFF3A3A6;">Compute losses</mark> all timesteps
	3. <mark style="background: #FFF3A3A6;">Backpropagate gradient contributions from last timestep</mark> to <mark style="background: #FFF3A3A6;">first</mark>
	4. <mark style="background: #FFF3A3A6;">Update shared weights</mark>
<mark style="background: #FFF3A3A6;">Gradient flow backward through time.</mark>


#### Training RNNs with cross-entropy loss
Each timestep $t$, predict **next word**:
$$\hat{y}_{t} = P(w_{t+1} \ | \ w_{1}, \dots , w_{t})$$
Each timestep $t$ model <mark style="background: #FFF3A3A6;">looks prev words</mark>, <mark style="background: #FFF3A3A6;">produces vector </mark>$z_t = Wh_t + b$, $z_t \in \mathbb{R}^{∣V∣}$, can be <mark style="background: #FFF3A3A6;">any real number,</mark> for poss words.<mark style="background: #FFF3A3A6;"> A softmax layer.</mark>
$$\hat{y}_t[i] = \frac{e^{z_{t}[i]}}{\sum_{j=1}^V e^{z_{t}[j]}}$$
<mark style="background: #FFF3A3A6;">converts raw score</mark> into <mark style="background: #FFF3A3A6;">pos number</mark>, <mark style="background: #FFF3A3A6;">preserving order</mark>, and <mark style="background: #FFF3A3A6;">yielding proper probability distribution. </mark>
We can say that:
$$P(w_{t+1}=i | w_{1} \dots w_{t}) = \frac {e^{(Wh_{t}+b)_{i}}} {\sum^V_{j=1}e^{(Wh_{t}+b)_{j}}}$$
<mark style="background: #FFF3A3A6;">Do for all i, </mark>and select highes probability according to z_t.

Because<mark style="background: #FFF3A3A6;"> correct next word known</mark> <mark style="background: #FFF3A3A6;">during training</mark>,<mark style="background: #FFF3A3A6;"> target distribution </mark>is **<mark style="background: #FFF3A3A6;">one-hot vector</mark>** over entire vocab; entry for correct word = 1, all others = 0. The **<mark style="background: #FFF3A3A6;">cross-entropy loss</mark>** for LMs is <mark style="background: #FFF3A3A6;">only determined</mark> by <mark style="background: #FFF3A3A6;">probability of next word</mark>, at timestep t, CE for one hot target is:
$$L_{CE}(\hat{y}_{t}, {y}_{t}) = -\log \hat{y}_{t}[w_{t+1}]$$
<mark style="background: #FFF3A3A6;">-log probability</mark> of <mark style="background: #FFF3A3A6;">actual known word </mark><mark style="background: #FFF3A3A6;">according to softmax layer and logit</mark>
No loss = 0, is perfect, if model predicts 1; <mark style="background: #FFF3A3A6;">penalised for assigning poor probability</mark> to correct word.


#### Training RNNs with teacher forcing
-<mark style="background: #FFF3A3A6;"> During training</mark>, model, <mark style="background: #FFF3A3A6;">predict next word</mark>, must determine, want to feed in prev prediction, as next input, or true prev word, from dataset, known during training.
- **Teacher forcing** - <mark style="background: #FFF3A3A6;">always feed model,</mark> correct <mark style="background: #FFF3A3A6;">prev 'gold' word.</mark>
- If feed model, own prediction, early mistakes, compound easily, can drift from correct context; enforce stable training, stay close, ground truth
	- <mark style="background: #FFF3A3A6;">Compute true loss</mark>, <mark style="background: #FFF3A3A6;">move next timestep</mark> <mark style="background: #FFF3A3A6;">using correct</mark> $w_{t+1}$ <mark style="background: #FFF3A3A6;">as input</mark> + prior hidden state as calculated, according, RNN arch. 

#### Feed-forward Neural LMs vs RNNs
- <mark style="background: #FFF3A3A6;">FFN-LMs,</mark> <mark style="background: #FFF3A3A6;">no memory</mark>, <mark style="background: #FFF3A3A6;">vectors = independent</mark>, no hidden state
- <mark style="background: #FFF3A3A6;">Limited</mark> by <mark style="background: #FFF3A3A6;">input size</mark>, determines context window
- <mark style="background: #FFF3A3A6;">Each pass of input</mark> = <mark style="background: #FFF3A3A6;">stateless,</mark> no internal compressed memory
- <mark style="background: #FFF3A3A6;">RNN</mark> capture<mark style="background: #FFF3A3A6;"> long-term dependencies;</mark> theoretically, start of text. 

#### Simple RNN-LM
- <mark style="background: #FFF3A3A6;">Process sequences</mark>, read <mark style="background: #FFF3A3A6;">one token at time</mark>, <mark style="background: #FFF3A3A6;">predict next token,</mark> <mark style="background: #FFF3A3A6;">progrssively enrich context</mark>, infinite n-gram LM.
### RNN challenges
- Theoretically, long term deps
- **<mark style="background: #FFF3A3A6;">Long dependencies</mark>**
	- <mark style="background: #FFF3A3A6;">Some words</mark>, <mark style="background: #FFF3A3A6;">depend</mark> on <mark style="background: #FFF3A3A6;">others</mark>, <mark style="background: #FFF3A3A6;">far away in text, </mark>regular RNN struggle, <mark style="background: #FFF3A3A6;">early influence fades</mark>, must acccomodate long contexts, architecture, <mark style="background: #FFF3A3A6;">only design hidden layers, accom so much</mark>.
- **Backpropagation through time**
	- <mark style="background: #FFF3A3A6;">Repeated multiplication</mark> of <mark style="background: #FFF3A3A6;">derivatives</mark> over many timesteps, cause problems, either <mark style="background: #FFF3A3A6;">vanihshing</mark>, or <mark style="background: #FFF3A3A6;">exploding gradient</mark>s. If vanuish, <mark style="background: #FFF3A3A6;">early timesteps</mark>, <mark style="background: #FFF3A3A6;">no influence,</mark> if <mark style="background: #FFF3A3A6;">explode, </mark><mark style="background: #FFF3A3A6;">unstable learning. </mark>
#### Solutions
-<mark style="background: #FFF3A3A6;"> Truncate BPTT</mark>, <mark style="background: #FFF3A3A6;">lose out long contexts</mark>, <mark style="background: #FFF3A3A6;">reduce vanishing</mark>/exploding effect
- <mark style="background: #FFF3A3A6;">Move more complex neural arch</mark>

### LSTMs, GRUs, Memnet
- **LSTMs**, <mark style="background: #FFF3A3A6;">introduce gates</mark>, <mark style="background: #FFF3A3A6;">control</mark>, <mark style="background: #FFF3A3A6;">what remember</mark>/forget
- **GRUs**, <mark style="background: #FFF3A3A6;">simple version of LSTMs</mark>
- **MEMnet**, <mark style="background: #FFF3A3A6;">permit explicit memory access</mark>
- **Transformers/attention**, <mark style="background: #FFF3A3A6;">directly connect, all pos in seq</mark>, <mark style="background: #FFF3A3A6;">solve long-range dep issues.</mark>

### RNN-based Neural lang models
- <mark style="background: #FFF3A3A6;">Train RNN</mark>, <mark style="background: #FFF3A3A6;">generate next tok</mark>
	- <mark style="background: #FFF3A3A6;">Model error</mark> via <mark style="background: #FFF3A3A6;">CE loss function</mark> <mark style="background: #FFF3A3A6;">targeting one-hot vector. </mark>
- Training, <mark style="background: #FFF3A3A6;">massive text coprora</mark>, <mark style="background: #FFF3A3A6;">learn lang patterns,</mark> no labelled data
- Allows, <mark style="background: #FFF3A3A6;">large scale</mark>, **<mark style="background: #FFF3A3A6;">self-supervised learning</mark>**
	- **SSL** machine learning method, model <mark style="background: #FFF3A3A6;">synthesise pseudo-labels</mark> from unlabelled data; model learn, by <mark style="background: #FFF3A3A6;">predicting part of input</mark>, from <mark style="background: #FFF3A3A6;">other parts</mark>
	- <mark style="background: #FFF3A3A6;">Next word prediction</mark> - task is <mark style="background: #FFF3A3A6;">predict next word,</mark> l<mark style="background: #FFF3A3A6;">abel comes from text itself. </mark>
	- Can<mark style="background: #FFF3A3A6;"> fine tune</mark> for specific tasks via supervised
	- Key insight to creation of <mark style="background: #FFF3A3A6;">foundation models</mark>
		- Large ML models, trained generally, then specialised, fine-tuned. 

#### Pre-training neural lang models
- <mark style="background: #FFF3A3A6;">Self-supervised pre-training</mark>, corpora
- Pretext tasks:
	- <mark style="background: #FFF3A3A6;">Masked lang modelling</mark>
		- "The cat MASK on the mat"
		- Task: predict masked word
		- Label: comes from sentence.
	- <mark style="background: #FFF3A3A6;">Next word prediction</mark>
- Pre-training, using self-supervised tasks, allow model<mark style="background: #FFF3A3A6;">, learn grammar</mark>, <mark style="background: #FFF3A3A6;">word meaning</mark>, <mark style="background: #FFF3A3A6;">context, world knowledge</mark>

### Fine-tuning NLMs
- Foundation model, adapt
- <mark style="background: #FFF3A3A6;">Add task-specific layer</mark> on top, e.g., <mark style="background: #FFF3A3A6;">classification head</mark> for sentiment analysis
- Decide, how train model, on new task
	1. <mark style="background: #FFF3A3A6;">Freeze</mark>
		- Keep pretrained LM params, only train new task-specific layers, faster training, less memory,<mark style="background: #FFF3A3A6;"> less risk overfitting</mark> if dataset small, but<mark style="background: #FFF3A3A6;"> low adaptability</mark>, <mark style="background: #FFF3A3A6;">result in lower perf,</mark> for <mark style="background: #FFF3A3A6;">divergent tasks. </mark>
	2. <mark style="background: #FFF3A3A6;">Tune</mark>
		- Update pretrained LMs along with new layers, adapt to task, <mark style="background: #FFF3A3A6;">better perf divergent tasks</mark>, <mark style="background: #FFF3A3A6;">overfit if data small</mark>, <mark style="background: #FFF3A3A6;">more memory </mark>and <mark style="background: #FFF3A3A6;">computation</mark> and training time. 