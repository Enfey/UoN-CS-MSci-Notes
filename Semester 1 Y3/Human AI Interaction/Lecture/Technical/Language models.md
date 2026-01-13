## Statistical Language Models
Before neural models, NLG used $n-gram$ probability models. Given a sequence of tokens, predict the next token. It assigns a probability to a sequence of words, and it helps decide what sounds natural in a language. 

For example, have two possible sentences
$P(\text{Nature is healing})$
$P(\text{Nature is killing})$

A good language model will assign $$P(\text{Nature is healing}) > P(\text{Nature is killing})$$
Before neural networks, language models were built on statistics from word counts. The probability of a whole sentence
$P(w_1, w_2, w_3,...w_n) = P(w_1) \times P(w_2∣w_1) \times P(w_3∣w_1, w_2) \times \dots \times P$ 
Impossible to compute directly. Aka:
$$\prod_iP(w_i ∣w_1, w_2 \dots w_{i-1})$$
We simplify this such that each word depends only on last few words, not entire sentence,
	$P(w_i∣w_{i-n+1}, \dots w_{i-1} )$ (range) 
Each word $i$ depends only on a **restricted set of preceding words.** 
Assumption: language is a **stochastic, memoryless process**. 
We use the above 2 statements to make the language countable.
	![](Pasted%20image%2020251224022448.png)
Example with $n=0$ 
	$𝑃(S_{1}) = 𝑃(𝑁𝑎𝑡𝑢𝑟𝑒) × 𝑃(𝑖𝑠) × 𝑃(ℎ𝑒𝑎𝑙𝑖𝑛𝑔)$
Example with $n = 1$ 
	$P(S_1) = 𝑃 (𝑁𝑎𝑡𝑢𝑟𝑒) × 𝑃 (𝑖𝑠 \ | \ 𝑁𝑎𝑡𝑢𝑟𝑒)× P( ℎ𝑒𝑎𝑙𝑖𝑛𝑔 \ | \ 𝑖𝑠)$ 
	
Counting these probabilities is much more feasible. 
$P(S_1) = P(Nature) \times \frac{\#(Nature \ is)}{\#(Nature)} \times \frac {\#(is \ healing)} {\#(is)}$

$P(w_i \ | \ w_{i-1}) = \frac{\#(w_{i-1}, w_i)}{\#(w_{i-1})}$ is the **Maximum Likelihood Estimate** (MLE).
### Example
![](Pasted%20image%2020251224022621.png)
Bigram counts, cell value  = $\#(w_{i-1}, w_i)$ 
![](Pasted%20image%2020251224022715.png)
Unigram counts. 

![](Pasted%20image%2020251224023042.png)
![](Pasted%20image%2020251224023108.png)
However, if any bigram has zero probability, then $P(sentence) = 0$, even if the sentence is grammatical; the combination has simply never appeared before, therefore MLE alone is insufficient. 

### Laplace smoothing
![](Pasted%20image%2020251224023536.png)
![](Pasted%20image%2020251224023617.png)

## Neural language models
Predict human language by assigning probabilities to word sequences based on computation, not counting. Neural networks are used to learn word representations and predict the next token directly. Explicit probability tables are replaced with **learned parameters**.

Statistical models suffered from sparsity, zero probabilities with parsing and subsequent need for smoothing methods, poor generalisation. Neural LMs solves this by embedding words in a continuous vector space and allowing similar words to have similar representations and generalising to unseen combinations. 

An **embedding** is a **dense numerical vector** representing a token. The embedding dimension $n$ determines $\mathbb{R}^n$.

Words like "car" and "bus" can be close in vector space, and the model can generalise, even if rarely seen, giving generalisation power beyond counts. 
![](Pasted%20image%2020251224030531.png)

The core issue is that neural networks typically require fixed-size input vectors, but natural language is inherently variable in length. One solution is to add special tokens to shorter sequences to normalise input length, and truncating for long sequences; this leads to arbitrary sequence length choice and loss of information. Another solution is to combine all word embeddings into one fixed size vector; like below
![](Pasted%20image%2020251224030839.png)
However, we lose word order. Sentences with the same words but completely different meaning are equivalent input under this scheme, leading to incorrect predictions. Fixes the size issue however.

The natural solution is to build a neural architecture that **naturally handles sequences**.

Examples:
- **Recurrent Neural Networks** (RNNs)
- **LSTMs** 
- **GRUs**
- **Transformers**
Such models process tokens sequentially and maintain internal state, encoding order and context. 

### Recurrent Neural Networks
An **RNN** is a neural network designed for sequential data, maintains a **recurrent** connection that lets information persist across time.

At each timestep $t$:
- Input: $x_t$ 
- Hidden state: $h_t$ 
- Output: $y_t$
The hidden state acts as **memory**.

RNNs natural model sequential probability via this architecture.

#### Input-Output Patterns
![](Pasted%20image%2020251224032532.png)

#### Elman vs Jordan Networks
Early RNN architectures. 
![](Pasted%20image%2020251224032727.png)
The hidden state update is $h_t = \sigma(W_xx_t + U_hh_{t-1} + b_h)$ 
The output computation is $y_t = \sigma(W_th_t + b_y)$,  "given current memory, what should I predict?"

![](Pasted%20image%2020251224033008.png)
Less expensive, but rarely used today. 

#### Unfolding an RNN through time 
An RNN over a sequence of length $T$ becomes a depth-$T$ neural network over time. Each layer essentially corresponds to one timestep. The same weights are used at every timestep; we unroll into $T$ copies of the same network, one for each time step, where in the unfolded view a chain of $T$ identical RNN cells exist where each cell receives input $x_t$ at time $t$, each cell receives the hidden state $h_{t-1}$ from the previous timestep, and each cell produces an output $y_t$ (useful for showing loss and identifying contriution) and passes its hidden state $h_t$ to the next cell.

Depending on task, outputs matter differently:
- **Many-to-many (sequence-to-sequence)**: Output at every timestep matters (e.g., part-of-speech tagging)
- **Many-to-one**: Only the final output $y_T$ matters (e.g., sentiment classification), usually only output at final step.
- **One-to-many**: Only initial input matters, but outputs at each timestep matter (e.g., image captioning)

Unrolling reveals that each hidden state depends on all previous inputs, and that earlier inputs influence later outputs indirectly. Each timestep uses the same parameters; the network does not learn different rules per position, which is why they are sequence models. The unrolling view is essential to see training difficulties.
#### Training RNNs: Backpropagation through time
Once unfolded, an RNN is just a very deep feedforward network. This permits **backpropagation through time**.
1. Foward pass through all timesteps
2. Compute losses at every time step
3. Backpropagate gradient contributions from last timestep to first
4. Update shared weights
Gradients flow backwards through time. 

#### Training RNNS with cross-entropy loss
At each timestep $t$, predict the **next word**:
$$\hat{y}_{t} = P(w_{t+1} \ | \ w_{1}, \dots , w_{t})$$
At each timestep $t$, the model looks at previous words and produces a vector: $z_{t} = Wh_{t}+b, z_{t} \in \mathbb{R}^{∣V∣}$ which can be any real number. A softmax layer:
$$\hat{y}_t[i] = \frac{e^{z_{t}[i]}}{\sum_{j=1}^V e^{z_{t}[j]}}$$converts the raw score into a positive number, preserving ordering and yielding a proper probability distribution. 
We can say that:
$$P(w_{t+1}=i | w_{1} \dots w_{t}) = \frac {e^{(Wh_{t}+b)_{i}}} {\sum^V_{j=1}e^{(Wh_{t}+b)_{j}}}$$
Because the correct next word is known during training, the target distribution is **one-hot** vector over the entire vocabulary, the entry for the correct word being 1, and all others being 0. The cross-entropy (CE) loss for LMs is **only** determined by the probability of next word. At timestep $t$, CE (for one-hot target) is:
$$L_{CE}(\hat{y}_{t}, {y}_{t}) = -\log \hat{y}_{t}[w_{t+1}]$$ 
The total loss is this summated according to $t$.

It should be noted that $\hat{y}_t$ is a a proper probability distribution when comparing to one-hot target vector. If the model predicts 1 for the correct word, there is no loss (=0) and is perfect. Penalises for assigning poor probability to correct word. 
#### Training RNNs with teacher forcing
- During training, model predicts next word at each timestep; must determine whether want to feed in previous prediction as next input, or the true previous word from the dataset known during training.
- In **teacher forcing**, we always feed the model the correct previous word (the 'gold' word).
- If we feed the model it's own prediction, early mistakes compound easily, and the model can drift from correct context; this way we enforce stable training by ensure stays close to ground truth.
	- At timestep $t$ input = correct word $w_t$ 
	- Hidden state = $h_{t-1}$ 
	- Output = predicted probability $\hat{y}_t = P(w_{t+1} \mid w_1, \dots, w_t)$
	- Compute loss with the **true next word** $w_{t+1}$
	- Move to next timestep using correct $w_{t+1}$ as input and prior hidden state as calculated depending on RNN architecture.

#### Feed-Forward Neural LMs vs RNN-LMs
- FFN-LMs have no memory, they process input vectors independently, and there is no hidden state that rolls over from one timestep to another.
- FFN-LMs are limited by their input size; their fixed input size directly determines the fixed context window size they have to determine $w_{t+1}$.
- Each pass of input to the model is stateless; there is no internal compressed memory to remember anything outside of the current context. 
- RNNs capture long-term dependencies, theoretically the context travels all the way to the beginning of the text.

#### A simple RNN LM
![](Pasted%20image%2020251225002347.png)

#### Challenges of recurrent neural networks
Theoretically, RNNs can capture **long-term dependencies** in sequences. However, in practice, they have serious challenges:

- **Long Dependencies**
	- Some words depend on others far away in the text. Regular RNNs struggle because the influence of early words fades over time. Must accommodate long contexts, architecture can only design hidden layers to accommodate so much for simple RNNs.
- **Backpropagation Through Time**
	- ![](Pasted%20image%2020251225010355.png)
Solutions do exist:
- Truncate BPTT and lose out on long contexts, reduces vanishing/exploding effect.
- Move to more complex neural architectures.
#### LSTMs, GRUs, MemNet
- LSTMs introduce gates to control what to remember or forget
- GRUs are a simpler version of LSTMs
- MEMnet allow explicit external memory access.
- Attention/Transformers directly connect all positions in a sequence to solve long-range dependency issues. 

#### RNN-based Neural Language Models
- Train the RNN to generate the next token
	- Model error via cross-entropy loss function targeting one-hot vector.
- By training on massive text corpora, models learn general language patterns without needing labeled data.
- Allows for large-scale **self-supervised learning**:
	- SSL is a type of machine learning where the model synthesises pseudo-labels from unlabelled data; the model learns by predicting part of the input from other parts.
	- Next word prediction - task is to predict next word, label comes automatically from the text itself. 
	- We can then fine-tune for specific tasks using supervised learning
- This is a key insight to the creation of **foundation models**
	- Large machine learning models trained on 'general' tasks, to be specialised later through fine-tuning
#### Pre-training neural language models
- Self-supervised pre-training on massive text data (no need for labels, typical bottleneck)
- Typical pretext tasks:
	- Masked language modelling
		- Input: “The cat MASK on the mat”
		- Task: predict the masked word → “sat”
		- Again, label comes from the sentence itself.
	- Next Word Prediction
		- Input: “The cat MASK on the mat”
		- Task: predict the masked word → “sat”
		- Again, label comes from the sentence itself.
- Pre-training using self-supervised tasks allows the model to learn grammar, word meanings, contextual relations, and some world knowledge
- These are general purpose representations which can be tightly specialised.
#### Fine-Tuning NLMs for specific tasks
- Once have a foundation model, can adapt for a specific downstream task. 
- Add a task-specific layer on top of it e.g., for sentiment analysis, add a classification head
- Decide how to train the model on this new task:
	1. Freeze the language model
		Keep all pretrained LM parameters fixed
		Only train the new task-specific layers on dataset
		Faster training, less memory and computation required
		Less risk of overfitting if dataset is small
		Model cannot adapt its internal representations for the task though, and may result in lower performance, especially for divergent tasks.
	2. Tune language model
		Update pretrained LM parameters along with the new layers
		Can adapt to task, and usually leads to better for performance for divergent tasks, but requires more memory and computation, has slower training time and can overfit if task-specific data is small.

