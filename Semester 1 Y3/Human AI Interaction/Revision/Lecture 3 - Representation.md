<mark style="background: #FFF3A3A6;">Before representation</mark>, <mark style="background: #FFF3A3A6;">raw text, </mark>undergoes, <mark style="background: #FFF3A3A6;">pre-processing steps</mark>, make consistent and structured. After this, have <mark style="background: #FFF3A3A6;">clean, structured tokens</mark>, can <mark style="background: #FFF3A3A6;">provide machine representation.</mark>

Computers, cannot interpret, natural lang, <mark style="background: #FFF3A3A6;">need numeric vectors.</mark>

A <mark style="background: #FFF3A3A6;">good representation should:</mark>
- **<mark style="background: #FFF3A3A6;">Capture semantics</mark>** - sim meaning, sim vectors
- **<mark style="background: #FFF3A3A6;">Preserve syntactic informatio</mark>n** - word order, relationships.
- **<mark style="background: #FFF3A3A6;">Handle information</mark>** - meaning changes contextually

## BOW representation
<mark style="background: #FFF3A3A6;">Use **vector space model**,</mark> <mark style="background: #FFF3A3A6;">represent text,</mark> such that <mark style="background: #FFF3A3A6;">vector space </mark>has<mark style="background: #FFF3A3A6;"> $V$ dimensions</mark>, where $V =\text{vocabulary size}$ . Document = vector in that space where $d_i = [w_1, w_2, ... w_v$]$ where $w_i$ is weight of word $i$ in document $i$.

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
	- $dist_{Euc}\left( A^{\to}B^{\to} = \sqrt{ \sum_{i=1}^{V}(A_{i}-B_{i})  }  \right)$
- Manhattan distance, measures absolute difference between word weights.
### Similarity in vector space
- How close/alike two vectors are, high value, more similar
- **Cosine similarity**
	- Measure angle between vectors, not their magnitude, two docs similar if point in same direction, even if one is longer
	- $cosine_{sim}(A^{\to}B^{\to}) = \frac {A^{\to} \cdot B^{\to}} {|| A^{\to}|| \ \ ||B^{\to}||}$
	- Measured from -1 to 1, 1 is identical direction, 0 is no shared terms, -1 is opposite direction. 

### Weighting functions
<mark style="background: #FFF3A3A6;">Given document $d$,</mark><mark style="background: #FFF3A3A6;"> each dimension of that vector</mark> <mark style="background: #FFF3A3A6;">computed from frequency</mark> of <mark style="background: #FFF3A3A6;">corresponding terms</mark> in that document. <mark style="background: #FFF3A3A6;">Raw frequency</mark> to weight, <mark style="background: #FFF3A3A6;">has problems</mark>, <mark style="background: #FFF3A3A6;">large docs</mark>, <mark style="background: #FFF3A3A6;">distant </mark>from <mark style="background: #FFF3A3A6;">short docs</mark>, <mark style="background: #FFF3A3A6;">even though similar.</mark>

Functions:
- <mark style="background: #FFF3A3A6;">Binary weighting</mark>
	- $weight(t,d) = \Bigg\{ 1 \ \text{if t} \in d, 0 \ otherwise$ 
- <mark style="background: #FFF3A3A6;"> Log weighting</mark>
	- $weight(t,d) = log(1+freq(t, d))$
	![](Pasted%20image%2020260110205831.png)
- <mark style="background: #FFF3A3A6;"> TF-IDF weighting</mark>
	- $weight(t,d) = log (1+freq(t,d)) + log(\frac{N}{n})$ 
	- Where:
		- $N$ = number of docs
		- $n$ = number of docs containing that term
		- $d$ = document
		- $t$ = term
	![[Pasted image 20251013220053.png]]
Can now represent any doc in fixed-size vec with appropriate weights; can be fed into machine learning algos.


## One-hot encoding
A <mark style="background: #FFF3A3A6;">one-hot </mark>is a <mark style="background: #FFF3A3A6;">group of bits</mark>, among which<mark style="background: #FFF3A3A6;"> legal combinations</mark> of <mark style="background: #FFF3A3A6;">values,</mark> are <mark style="background: #FFF3A3A6;">those with a single high bit</mark>, and <mark style="background: #FFF3A3A6;">all others low.</mark>

$L \times V$<mark style="background: #FFF3A3A6;"> dimensions</mark> where $L$ = length of input and $V$ = vocab size

Usually 1 high bit per column vector corresponding to word at that input step.

Example:
- $d_1$ "The red cat"
- $d_2$ "The red dog"
- $d_3$ "Cat, red cat and dog"
- $V = |\{the, red, cat, dog, and\}|$
- ![[Pasted image 20251014001739.png]]
Assumption:<mark style="background: #FFF3A3A6;"> all words equally unrelated,</mark> no semantics, <mark style="background: #FFF3A3A6;">large sparse matrices</mark>, do <mark style="background: #FFF3A3A6;">not encode semantics.</mark>


## Dense Vector Representations
<mark style="background: #FFF3A3A6;">General term,</mark><mark style="background: #FFF3A3A6;"> representing data</mark>, as l<mark style="background: #FFF3A3A6;">ow-dimensional</mark>, r<mark style="background: #FFF3A3A6;">eal-valued vector</mark> (<mark style="background: #FFF3A3A6;">almost all elements non-zero</mark>). Goal = <mark style="background: #FFF3A3A6;">encode meaningful </mark><mark style="background: #FFF3A3A6;">similarity</mark>. <mark style="background: #FFF3A3A6;">Sim items</mark> have <mark style="background: #FFF3A3A6;">vectors close together</mark> in <mark style="background: #FFF3A3A6;">vector space..</mark>

### Word embeddings
<mark style="background: #FFF3A3A6;">Specific type</mark>, <mark style="background: #FFF3A3A6;">dense vector representation</mark>, <mark style="background: #FFF3A3A6;">applied to words in NLP</mark>. Represent <mark style="background: #FFF3A3A6;">each word</mark> as<mark style="background: #FFF3A3A6;"> dense vector</mark> <mark style="background: #FFF3A3A6;">learned from data</mark>, such that <mark style="background: #FFF3A3A6;">semantic relationships</mark> are <mark style="background: #FFF3A3A6;">captured.</mark>

Let $v(x)$ be the embedding vector of word $x$
**Similarity:** $Similarity(v(dog), v(cat))>Simiarlity(v(dog), v(car))$
**Analogy:** $v(king) - v(man) + v(woman) ≈ v(queen)$

<mark style="background: #FFF3A3A6;">Properties</mark> <mark style="background: #FFF3A3A6;">depend </mark>on <mark style="background: #FFF3A3A6;">how embeddings</mark> were<mark style="background: #FFF3A3A6;"> computed</mark>.

<mark style="background: #FFF3A3A6;">Mathematical intuition</mark> is via <mark style="background: #FFF3A3A6;">co-occurrence</mark>, by <mark style="background: #FFF3A3A6;">defining context</mark> <mark style="background: #FFF3A3A6;">window</mark> of size $n$ for target word $w_t$, a model<mark style="background: #FFF3A3A6;"> can learn</mark> <mark style="background: #FFF3A3A6;">which words</mark>, <mark style="background: #FFF3A3A6;">appear near each other,</mark> <mark style="background: #FFF3A3A6;">construct</mark> <mark style="background: #FFF3A3A6;">co-occurrence matrix</mark> $M$, <mark style="background: #FFF3A3A6;">factorise</mark> this into <mark style="background: #FFF3A3A6;">dense vector.</mark>
- Example:
	- The Cat sat on the mat
	- Context window size =2
	- Context for "sat" = "the, cat, on, the"
	- Context for mat = "on, the"
<mark style="background: #FFF3A3A6;">Numerous ways</mark>, compute embeddings, but <mark style="background: #FFF3A3A6;">context window</mark> = <mark style="background: #FFF3A3A6;">universal.</mark> **GloVe** combines co-occurrence matrices with local context window learning. 


### Word2Vec
<mark style="background: #FFF3A3A6;">Shallow neural network</mark>, <mark style="background: #FFF3A3A6;">learns word vectors</mark>, by <mark style="background: #FFF3A3A6;">predicting context </mark>(given window size $n$).

Two arch:
1. **CBOW**
	<mark style="background: #FFF3A3A6;">Predict target word</mark> <mark style="background: #FFF3A3A6;">given</mark> its <mark style="background: #FFF3A3A6;">context.</mark>
	$P(w_t|context) = P(w_t|w_{t-2},w_{t-1}, w_{t+1}, w_{t+2})$ 
2. **Skip-gram**
	<mark style="background: #FFF3A3A6;">Predict</mark> surrounding words/<mark style="background: #FFF3A3A6;">context </mark><mark style="background: #FFF3A3A6;">given target word</mark>
	$P(context|w_t)$ 
	For $n = 2$ <mark style="background: #FFF3A3A6;">create training pairs</mark> (target, context), <mark style="background: #FFF3A3A6;">given the target word</mark> $x$ , the word $y$ appears in its context<mark style="background: #FFF3A3A6;">. Simple shallow neural </mark>network taking <mark style="background: #FFF3A3A6;">one-hot vector of target word,</mark> <mark style="background: #FFF3A3A6;">hidden layer</mark>, and <mark style="background: #FFF3A3A6;">output layer</mark>, <mark style="background: #FFF3A3A6;">softmax over all word</mark>s to<mark style="background: #FFF3A3A6;"> predict context</mark>. Each word has <mark style="background: #FFF3A3A6;">two vec representations,</mark> <mark style="background: #FFF3A3A6;">input and output vector.</mark>

Acts like form of pre-training, embedding is trained on data, and that embedding its used to train another model. 


## Representation and Similarity
### Components of a basic search engine
- <mark style="background: #FFF3A3A6;">Search engine</mark> consists of:
	- **<mark style="background: #FFF3A3A6;">Corpus</mark>** - Collection of docs, represented, shared feature space
	- **<mark style="background: #FFF3A3A6;">Relevance function</mark>** - measure how relevant doc is to query, between 0 and 1
		- <mark style="background: #FFF3A3A6;">Input query</mark>,<mark style="background: #FFF3A3A6;"> output relevance score</mark>
	- **<mark style="background: #FFF3A3A6;">UI</mark>** - allow user to enter queries
	- **<mark style="background: #FFF3A3A6;">Ranking interface</mark>** - order docs
### Search as similarity
- <mark style="background: #FFF3A3A6;">Search</mark>, an be <mark style="background: #FFF3A3A6;">viewed,</mark> <mark style="background: #FFF3A3A6;">similarity computation</mark> between query and docs.
$$Relevance(query, document) = Similarity(query, document)$$
- <mark style="background: #FFF3A3A6;">Various models approximate</mark> this<mark style="background: #FFF3A3A6;"> similarity</mark>:
	- **<mark style="background: #FFF3A3A6;">Probablistic models</mark>**
		- Estimate likelihood
	- **<mark style="background: #FFF3A3A6;">Vector space models</mark>**
		- Cosine similarity, query vec
	- **<mark style="background: #FFF3A3A6;">Fuzzy models</mark>**
		- Partial match
	- **<mark style="background: #FFF3A3A6;">Boolean models</mark>**
		- Exact match
### Classification as similarity
<mark style="background: #FFF3A3A6;">Given labelled corpus</mark> and <mark style="background: #FFF3A3A6;">new doc</mark> $d$
1. <mark style="background: #FFF3A3A6;">Find</mark> the $K$ <mark style="background: #FFF3A3A6;">most sim docs</mark> to $d$ 
2. <mark style="background: #FFF3A3A6;">Assign</mark> he <mark style="background: #FFF3A3A6;">majority label</mark> <mark style="background: #FFF3A3A6;">amongst</mark> these <mark style="background: #FFF3A3A6;">neighbours.</mark>
<mark style="background: #FFF3A3A6;">BOW representation</mark> often used, <mark style="background: #FFF3A3A6;">measuring similarities,</mark> <mark style="background: #FFF3A3A6;">choosing</mark> $K$ <mark style="background: #FFF3A3A6;">critical</mark> to <mark style="background: #FFF3A3A6;">success</mark>, label new instances by comparing to existing.


