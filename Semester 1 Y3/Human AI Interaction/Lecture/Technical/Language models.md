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
$P(w_i∣w_{i-n+1}, \dots w_{i-1})$ 
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

The natural solution is to build a neural architecture athat naturally handles sequences. 



