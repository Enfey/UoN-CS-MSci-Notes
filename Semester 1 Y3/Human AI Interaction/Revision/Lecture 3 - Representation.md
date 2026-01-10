Before representation, raw text, undergoes, pre-processing steps, make consistent and structured. After this, have clean, structured tokens, can provide machine representation.

Computers, cannot interpret, natural lang, need numeric vectors.

A good representation should:
- **Capture semantics** - sim meaning, sim vectors
- **Preserve syntactic information** - word order, relationships.
- **Handle information** - meaning changes contextually

## BOW representation
Use **vector space model**, represent text, such that vector space has $V$ dimensions, where $V =\text{vocabulary size}$ . Document = vector in that space where $d_i = [w_1, w_2, ... w_v$]$ where $w_i$ is weight of word $i$ in document $i$.

Transforms text corpus into **document-term matrix** $X \in R^{D\times V}$ where:
- $D$ is number of documents
- $V$ is vocab size
- $w_{ij}$ is frequency/weight of word $j$ in document $i$ 
$$
X =
\begin{bmatrix}
x_{11} & x_{12} & \cdots & x_{1V} \\
x_{21} & x_{22} & \cdots & x_{2V} \\
\vdots & \vdots & \ddots & \vdots \\
x_{D1} & x_{D2} & \cdots & x_{DV}
\end{bmatrix}
$$
## Distance in vector space
- How far apart two vectors are, low value = more similar
- Euclidean distance
	- Measure straight-line distance between two vectors in $V$ dimensional space
	- $dist_{Euc}\left( A^{\to}B^{\to} = \sqrt{ \sum_{i=1}^{V}(A_{i}B_{i})  }  \right)$
- Manhattan distance, measures absolute difference between word weights.
### Similarity in vector space
- How close/alike two vectors are, high value, more similar
- **Cosine similarity**
	- Measure angle between vectors, not their magnitude, two docs similar if point in same direction, even if one is longer
	- $cosine_{sim}(A^{\to}B^{\to}) = \frac {A^{\to} \cdot B^{\to}} {|| A^{\to}|| \ \ ||B^{\to}||}$
	- Measured from -1 to 1, 1 is identical direction, 0 is no shared terms, -1 is opposite direction. 

### Weighting functions
Given document $d$, each dimension of that vector computed from frequency of corresponding terms in that document. Raw frequency to weight, has problems, large docs, distant from short docs, even though similar.

Functions:
- Binary weighting
	- $weight(t,d) = \Bigg\{ 1 \ \text{if t} \in d, 0 \ otherwise$ 
- Log weighting
	- $weight(t,d) = log(1+freq(t, d))$
	![](Pasted%20image%2020260110205831.png)
- TF-IDF weighting
	- $weight(t,d) = log (1+freq(t,d)) + log(\frac{N}{n})$ 
	- Where:
		- $N$ = number of docs
		- $n$ = number of docs containing that term
		- $d$ = document
		- $t$ = term
	![[Pasted image 20251013220053.png]]
Can now represent any doc in fixed-size vec with appropriate weights; can be fed into machine learning algos.


## One-hot encoding
A one-hot is a group of bits, among which legal combinations of values, are those with a single high bit, and all others low.

$L \times V$ dimensions where $L$ = length of input and $V$ = vocab size

Usually 1 high bit per column vector corresponding to word at that input step.

Example:
- $d_1$ "The red cat"
- $d_2$ "The red dog"
- $d_3$ "Cat, red cat and dog"
- $V = |\{the, red, cat, dog, and\}|$
- ![[Pasted image 20251014001739.png]]
Assumption: all words equally unrelated, no semantics, large sparse matrices, do not encode semantics.


## Dense Vector Representations
General term, representing data, as low-dimensional, real-valued vector (almost all elements non-zero). Goal = encode meaningful similarity. Sim items have vectors close together in vector space..

### Word embeddings
Specific type, dense vector representation, applied to words in NLP. Represent each word as dense vector learned from data, such that semantic relationships are captured.

Let $v(x)$ be the embedding vector of word $x$
**Similarity:** $Similarity(v(dog), v(cat))>Simiarlity(v(dog), v(car))$
**Analogy:** $v(king) - v(man) + v(woman) ≈ v(queen)$

Properties depend on how embeddings were computed.

Mathematical intuition is via co-occurrence, by defining context window of size $n$ for target word $w_t$, a model can learn which words, appear near each other, construct co-occurrence matrix $M$, factorise this into dense vector.
- Example:
	- The Cat sat on the mat
	- Context window size =2
	- Context for "sat" = "the, cat, on, the"
	- Context for mat = "on, the"
Numerous ways, compute embeddings, but context window = universal. **GloVe** combines co-occurrence matrices with local context window learning. 


### Word2Vec
Shallow neural network, learns word vectors, by predicting context (given window size $n$).

Two arch:
1. **CBOW**
	Predict target word given its context.
	$P(w_t|context) = P(w_t|w_{t-2},w_{t-1}, w_{t+1}, w_{t+2})$ 
2. **Skip-gram**
	Predict surrounding words/context given target word
	$P(context|w_t)$ 
	For $n = 2$ create training pairs (target, context), given the target word $x$ , the word $y$ appears in its context. Simple shallow neural network taking one-hot vector of target word, hidden layer, and output layer, softmax over all words to predict context. Each word has two vec representations, input and output vector.

Acts like form of pre-training, embedding is trained on data, and that embedding its used to train another model. 


## Representation and Similarity
### Components of a basic search engine
- Search engine consists of:
	- **Corpus** - Collection of docs, represented, shared feature space
	- **Relevance function** - measure how relevant doc is to query, between 0 and 1
		- Input query, output relevance score
	- **UI** - allow user to enter queries
	- **Ranking interface** - order docs
### Search as similarity
- Search, an be viewed, similarity computation between query and docs.
$$Relevance(query, document) = Similarity(query, document)$$
- Various models approximate this similarity:
	- **Probablistic models**
		- Estimate likelihood
	- **Vector space models**
		- Cosine similarity, query vec
	- **Fuzzy models**
		- Partial match
	- **Boolean models**
		- Exact match
### Classification as similarity
Given labelled corpus and new doc $d$
1. Find the $K$ most sim docs to $d$ 
2. Assign he majority label amongst these neighbours.
BOW representation often used, measuring similarities, choosing $K$ critical to success, label new instances by comparing to existing.


