## Term-document matrix 
**Document-term matrix** $X \in R^{D\times V}$ where:
- $D$ is the number of documents
- $V$ is the vocabulary size
- $x_{ij}$ is the frequency of a word $j$ in a document $i$ 
$$
X =
\begin{bmatrix}
x_{11} & x_{12} & \cdots & x_{1V} \\
x_{21} & x_{22} & \cdots & x_{2V} \\
\vdots & \vdots & \ddots & \vdots \\
x_{D1} & x_{D2} & \cdots & x_{DV}
\end{bmatrix}
$$
Storing everything here, permit useful ops, e.g., compute sim; can be used to cluster, rank, compare.


### The query
Query = another document, mapped to vector space induced from document collection. 

### Building inverted matrix
DS underlying search engines = term-document matrix. Simply the transpose of the document-term matrix. 
$$
X =
\begin{bmatrix}
x_{11} & x_{12} & \cdots & x_{1D} \\
x_{21} & x_{22} & \cdots & x_{2D} \\
\vdots & \vdots & \ddots & \vdots \\
x_{V1} & x_{V2} & \cdots & x_{VD}
\end{bmatrix}
$$
Now ask what documents contain a word, rather than what words are in document. 

Each term essentially becomes key in dictionary, each term points to list of docs where it appears. This is known as an **inverted index**.
	**Vocabulary** - all unique terms appear in corpus
	**Term-Document matrix** - every term in dictionary, associated list of docs containing that term. 
	![](Pasted%20image%2020260111013524.png)
	Can enrich  postings with more than document id, e.g., term frequency(as above), positional information. 

### Query processing with inverted index and matrix
Efficiency gain clear during query processing. To find doc matching multi-word query, look up stemmed/lemmatised terms and retrieve postings list and compute intersection to yield IDs of docs in both, achievable in linear time. 


## Relevance and Similarity
Key function info retrieval system, compute relevance of doc with respect to query. Because query = small doc, can approx relevance via similarity measure. 

Goal of search engine = rank docs according to decreasing relevance where relevance will be approximated as **cosine similarity** or another mechanism. 


### Cosine similarity
Between query document $q$ and document $d$ is cosine of the angle between the document vector and query vector in the vector space as induced by the vocabulary. Defined as:
$$
\text{similarity}(q, d) = 
\frac{q \cdot d}{\|q\| \, \|d\|} =
\frac{\sum_{i=1}^{n} q_i d_i}{
\sqrt{\sum_{i=1}^{n} q_{i^2}} \, 
\sqrt{\sum_{i=1}^{n} d_{i^2}}}

$$
$q$ must undergo same weighting as document $d$ for fair similarity computation e.g., if binary, then must do binary and then compute, probably in exam just using raw or binary. 

## The search
Now have relevance function via cosine similarity, inverted index by way of term-document matrix. 

### Algo indexing
1. Compute vocab of corpus
2. Use vocab to compute term-document matrix
3. Apply term weighting to resultant term-document matrix
### Algo search
1. Preprocess search query
2. Map search query onto document collection-induced vector space
3. Apply term weighting technique to the vector features.
4. Compute cosine similarity with search query for each document in collection
5. Rank documents by decreasing similarity

### Confidence threshold + fallback
Key concept HAII, management of failure, always chance query does not match anything, usually computed by some sort of arbitrary threshold in our similarity under which nothing is considered relevant. 

#### Fallback by warning message
Display nothing relevant, suggest reformulation
#### Fallback by query expansion
One may use lang model e.g., given existing query, what is most likely next term entered by user, or may use semantic resource e.g., ontology to refine the query. 
Adding something to query, should not take over from what the user entered. 


## Evaluating search results
How does one know if search algorithm is good? Several metrics


### Human assessment of relevance

### Precision

### Recall

### Normalised Discounted Cumulative Gain



