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
Measure quality of search engine, based on set of reference queries, which know expected results. Results can be binary or graded (0 to 2).

A sample query would be represented as:
$$q_1 = \{d_{1} : 2, d_{5} : 2, d_{6}, : 2, d_{7}, d_{8} : 1\}$$
Assumption of 0 for all other docs, knowing this, can compute several metrics. 
### Precision
Precision$@ K$ Concerned, how many of the returned $K$ documents are relevant, focus only top $K$ results e.g., first page of search results. Usually evaluated with binary judgements, no grading of relevance. 
$$Precision = \frac{|\{relevant documents\} \cap \{retrieved documents\}| }{|\{retrieved documents\}|}$$

### Recall
Recall$@K$ concerned with how many of the relevant documents have been retrieved. Common to plot with multiple $K$ values to see how results of algo change as more results returned. Makes sense, small, specialised search engines, where restricted set of docs deemed relevant.
$$\frac{|\{relevant \ documents\} \cap \{retrieved \ documents\}|} {|\{relevant \ documents\}|}$$

### NDCG
Normalised Discounted Cumulative Gain$@p$;way to evaluate search results with graded relevance. 

Sum up relevance scores of top $P$ results
$$CG@P=\sum_{i=1}^{P}rel_{i}$$
Then discount lower ranks logarithmically
$$DCG@P\sum_{i=1}^{P}\frac{2^{rel_{i}}-1}{\log_{2}(i+1)}$$
We compute the best possible DCG i.e., the DCG if we had ranked all relevant documents perfectly.
$$IDCG@K = DGC@K$$
To make results comparable across queries or systems, we divide the actual $DCG$ by the ideal $DCG$


#### Evaluation Campaigns
Organised competitions, research groups test their search or ranking algorithms on shared datasets used shared evaluation metrics. To evaluate a search engine, one must know which documents are relevant to each query. But when dealing with billions of documents, cannot manually label all of them. Use a shortcut called pooling:
1. Each search system runs a fixed set of queries called evaluation topics, each system returns the top $N$ results for each query. 
2. A central authority takes the union of all documents returned during that search
3. The union is randomly split and assigned for labelling
4. Each participant receives a subset of the pool and labels each document as relevant, partially relevant, not relevant
5. After labelling, all judgments are aggregated so that each document gets a final, consensus based relevance label.

