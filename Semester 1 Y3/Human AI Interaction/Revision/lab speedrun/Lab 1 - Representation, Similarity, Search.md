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
<mark style="background: #FFF3A3A6;">Storing everything here, permit useful ops, e.g., compute sim; can be used to cluster, rank, compare.</mark>


### The query
<mark style="background: #FFF3A3A6;">Query</mark> = <mark style="background: #FFF3A3A6;">another document,</mark> <mark style="background: #FFF3A3A6;">mapped</mark> to <mark style="background: #FFF3A3A6;">vector space</mark> induced from document collection. 

### Building inverted matrix
<mark style="background: #FFF3A3A6;">DS</mark> <mark style="background: #FFF3A3A6;">underlying</mark><mark style="background: #FFF3A3A6;"> search engines</mark> = <mark style="background: #FFF3A3A6;">term-document matrix</mark>. Simply the <mark style="background: #FFF3A3A6;">transpose</mark> of the <mark style="background: #FFF3A3A6;">document-term matrix. </mark>
$$
X =
\begin{bmatrix}
x_{11} & x_{12} & \cdots & x_{1D} \\
x_{21} & x_{22} & \cdots & x_{2D} \\
\vdots & \vdots & \ddots & \vdots \\
x_{V1} & x_{V2} & \cdots & x_{VD}
\end{bmatrix}
$$
Now <mark style="background: #FFF3A3A6;">ask what documents</mark> <mark style="background: #FFF3A3A6;">contain</mark> a <mark style="background: #FFF3A3A6;">word</mark>, <mark style="background: #FFF3A3A6;">rather</mark> than <mark style="background: #FFF3A3A6;">what words</mark> are<mark style="background: #FFF3A3A6;"> in document.</mark> 

<mark style="background: #FFF3A3A6;">Each term</mark> essentially <mark style="background: #FFF3A3A6;">becomes key</mark> in <mark style="background: #FFF3A3A6;">dictionary</mark>, each term <mark style="background: #FFF3A3A6;">points to</mark> list of <mark style="background: #FFF3A3A6;">docs </mark><mark style="background: #FFF3A3A6;">where it appears</mark>. This is known as an **<mark style="background: #FFF3A3A6;">inverted index</mark>**.
	**<mark style="background: #FFF3A3A6;">Vocabulary</mark>** - <mark style="background: #FFF3A3A6;">all unique terms appear in corpus</mark>
	**Term-Document matrix** - <mark style="background: #FFF3A3A6;">every term in dictionary</mark>, <mark style="background: #FFF3A3A6;">associated list of docs containing that term. </mark>
	![](Pasted%20image%2020260111013524.png)
	<mark style="background: #FFF3A3A6;">Can enrich  postings</mark> with <mark style="background: #FFF3A3A6;">more than document id</mark>, e.g.,<mark style="background: #FFF3A3A6;"> term frequency(</mark>as above), <mark style="background: #FFF3A3A6;">positional information</mark>. 

### Query processing with inverted index and matrix
<mark style="background: #FFF3A3A6;">Efficiency gain</mark> <mark style="background: #FFF3A3A6;">clear</mark> during query processing. To <mark style="background: #FFF3A3A6;">find doc matching multi-word query</mark>, <mark style="background: #FFF3A3A6;">look up stemmed</mark><mark style="background: #FFF3A3A6;">/lemmatised terms</mark> and <mark style="background: #FFF3A3A6;">retrieve postings list</mark> and <mark style="background: #FFF3A3A6;">compute intersection</mark> to <mark style="background: #FFF3A3A6;">yield IDs</mark> of <mark style="background: #FFF3A3A6;">docs</mark> <mark style="background: #FFF3A3A6;">in both,</mark> <mark style="background: #FFF3A3A6;">achievable</mark> in <mark style="background: #FFF3A3A6;">linear time.</mark> 


## Relevance and Similarity
<mark style="background: #FFF3A3A6;">Key function</mark> <mark style="background: #FFF3A3A6;">info retrieval system</mark>,<mark style="background: #FFF3A3A6;"> compute relevance</mark> of <mark style="background: #FFF3A3A6;">doc</mark> with respect to <mark style="background: #FFF3A3A6;">query</mark>. Because query = small doc, can<mark style="background: #FFF3A3A6;"> approx relevance</mark> via <mark style="background: #FFF3A3A6;">similarity measure. </mark>

<mark style="background: #FFF3A3A6;">Goa</mark><mark style="background: #FFF3A3A6;">l</mark> of<mark style="background: #FFF3A3A6;"> search engine</mark> = <mark style="background: #FFF3A3A6;">rank docs</mark> according to d<mark style="background: #FFF3A3A6;">ecreasing relevance</mark> where<mark style="background: #FFF3A3A6;"> relevance</mark> will be <mark style="background: #FFF3A3A6;">approximated </mark>as **<mark style="background: #FFF3A3A6;">cosine similarity</mark>** or another mechanism. 


### Cosine similarity
<mark style="background: #FFF3A3A6;">Between</mark> <mark style="background: #FFF3A3A6;">query document</mark> $q$ and document $d$ is <mark style="background: #FFF3A3A6;">cosine</mark> of the <mark style="background: #FFF3A3A6;">angle between the document vector and query vector</mark> in the vector space as induced by the vocabulary. <mark style="background: #FFF3A3A6;">Defined as</mark>:
$$
\text{similarity}(q, d) = 
\frac{q \cdot d}{\|q\| \, \|d\|} =
\frac{\sum_{i=1}^{n} q_i d_i}{
\sqrt{\sum_{i=1}^{n} q_{i^2}} \, 
\sqrt{\sum_{i=1}^{n} d_{i^2}}}

$$
$q$ <mark style="background: #FFF3A3A6;">must undergo</mark> <mark style="background: #FFF3A3A6;">same weighting</mark> as <mark style="background: #FFF3A3A6;">document</mark> $d$ for <mark style="background: #FFF3A3A6;">fair similarity </mark><mark style="background: #FFF3A3A6;">computation</mark> e.g., if binary, then must do binary and then compute, probably in exam just using raw or binary. 

## The search
<mark style="background: #FFF3A3A6;">Now</mark> have <mark style="background: #FFF3A3A6;">relevance function</mark> via cosine similarity, <mark style="background: #FFF3A3A6;">inverted index</mark> by way of term-document matrix. 

### Algo indexing
1. <mark style="background: #FFF3A3A6;">Compute vocab</mark> of corpus
2. <mark style="background: #FFF3A3A6;">Use vocab</mark> to <mark style="background: #FFF3A3A6;">compute</mark> <mark style="background: #FFF3A3A6;">term-document matrix</mark>
3. <mark style="background: #FFF3A3A6;">Apply term weighting</mark> to <mark style="background: #FFF3A3A6;">resultant</mark> <mark style="background: #FFF3A3A6;">term-document matrix</mark>
### Algo search
1. <mark style="background: #FFF3A3A6;">Preprocess</mark> search <mark style="background: #FFF3A3A6;">query</mark>
2. <mark style="background: #FFF3A3A6;">Map</mark> search<mark style="background: #FFF3A3A6;"> query</mark> onto <mark style="background: #FFF3A3A6;">document collection-induced vector space</mark>
3. <mark style="background: #FFF3A3A6;">Apply term weighting technique</mark> to the <mark style="background: #FFF3A3A6;">vector features.</mark>
4. <mark style="background: #FFF3A3A6;">Compute cosine similarity</mark> with <mark style="background: #FFF3A3A6;">search query </mark>for<mark style="background: #FFF3A3A6;"> each document</mark> in <mark style="background: #FFF3A3A6;">collection</mark>
5. <mark style="background: #FFF3A3A6;">Rank documents</mark> by <mark style="background: #FFF3A3A6;">decreasing similarity</mark>

### Confidence threshold + fallback
Key concept HAII, management of failure, always chance query does not match anything, <mark style="background: #FFF3A3A6;">usually computed</mark> by some sort of <mark style="background: #FFF3A3A6;">arbitrary threshold</mark> in our similarity <mark style="background: #FFF3A3A6;">under which</mark> <mark style="background: #FFF3A3A6;">nothing</mark> is <mark style="background: #FFF3A3A6;">considered relevant. </mark>

#### <mark style="background: #FFF3A3A6;"> Fallback by warning message</mark>
Display nothing relevant, suggest reformulation
#### <mark style="background: #FFF3A3A6;">Fallback by query expansion</mark>
One may use lang model e.g., given existing query, what is <mark style="background: #FFF3A3A6;">most likely next term entered by user</mark>, or may <mark style="background: #FFF3A3A6;">use semantic resource</mark> e.g., <mark style="background: #FFF3A3A6;">ontology</mark> to refine the query. 
Adding something to query, s<mark style="background: #FFF3A3A6;">hould not take over from what the user entered. </mark>


## Evaluating search results
How does one know if search algorithm is good? Several metrics


### Human assessment of relevance
<mark style="background: #FFF3A3A6;">Measure quality</mark> of <mark style="background: #FFF3A3A6;">search engine,</mark><mark style="background: #FFF3A3A6;"> based</mark> on set of <mark style="background: #FFF3A3A6;">reference queries</mark>, which <mark style="background: #FFF3A3A6;">know</mark> <mark style="background: #FFF3A3A6;">expected results.</mark> <mark style="background: #FFF3A3A6;">Results</mark> can be <mark style="background: #FFF3A3A6;">binary</mark> or <mark style="background: #FFF3A3A6;">graded </mark>(0 to 2).

A<mark style="background: #FFF3A3A6;"> sample query</mark> would be represented as:
$$q_1 = \{d_{1} : 2, d_{5} : 2, d_{6}, : 2, d_{7}, d_{8} : 1\}$$
<mark style="background: #FFF3A3A6;">Assumption</mark> of <mark style="background: #FFF3A3A6;">0</mark> for<mark style="background: #FFF3A3A6;"> all other docs</mark>,<mark style="background: #FFF3A3A6;"> knowing this</mark>, can c<mark style="background: #FFF3A3A6;">ompute several metrics. </mark>
### Precision
Precision$@ K$ <mark style="background: #FFF3A3A6;">Concerned</mark>,<mark style="background: #FFF3A3A6;"> how many</mark> of the<mark style="background: #FFF3A3A6;"> returned</mark> $K$ <mark style="background: #FFF3A3A6;">documents</mark> are <mark style="background: #FFF3A3A6;">relevant,</mark> focus only <mark style="background: #FFF3A3A6;">top</mark> $K$ <mark style="background: #FFF3A3A6;">results</mark> e.g., first page of search results. Usually<mark style="background: #FFF3A3A6;"> evaluated</mark> with <mark style="background: #FFF3A3A6;">binary judgements</mark>, no <mark style="background: #FFF3A3A6;">grading of relevance. </mark>
$$Precision = \frac{|\{relevant documents\} \cap \{retrieved documents\}| }{|\{retrieved documents\}|}$$

### Recall
Recall$@K$ <mark style="background: #FFF3A3A6;">concerned</mark> with <mark style="background: #FFF3A3A6;">how many</mark> of the <mark style="background: #FFF3A3A6;">relevant documents</mark> have<mark style="background: #FFF3A3A6;"> been retrieved</mark>. Common to <mark style="background: #FFF3A3A6;">plot</mark> with <mark style="background: #FFF3A3A6;">multiple</mark> $K$ <mark style="background: #FFF3A3A6;">values</mark> to <mark style="background: #FFF3A3A6;">see</mark> how <mark style="background: #FFF3A3A6;">results</mark> of <mark style="background: #FFF3A3A6;">algo change</mark> as<mark style="background: #FFF3A3A6;"> more results</mark> <mark style="background: #FFF3A3A6;">returned</mark>. Makes sense, <mark style="background: #FFF3A3A6;">small,</mark> <mark style="background: #FFF3A3A6;">specialised search engines</mark>, where restricted set of docs deemed relevant.
$$\frac{|\{relevant \ documents\} \cap \{retrieved \ documents\}|} {|\{relevant \ documents\}|}$$

### NDCG
<mark style="background: #FFF3A3A6;">Normalised</mark> <mark style="background: #FFF3A3A6;">Discounted</mark><mark style="background: #FFF3A3A6;"> Cumulative Gain</mark>$@p$;way to e<mark style="background: #FFF3A3A6;">valuate search </mark><mark style="background: #FFF3A3A6;">results</mark> with <mark style="background: #FFF3A3A6;">graded relevance</mark>. 

<mark style="background: #FFF3A3A6;">Sum up</mark> <mark style="background: #FFF3A3A6;">relevance scores</mark> of <mark style="background: #FFF3A3A6;">top</mark> $P$ <mark style="background: #FFF3A3A6;">results</mark>
$$CG@P=\sum_{i=1}^{P}rel_{i}$$
Then <mark style="background: #FFF3A3A6;">discount</mark> <mark style="background: #FFF3A3A6;">lower ranks</mark><mark style="background: #FFF3A3A6;"> logarithmically</mark>
$$DCG@P\sum_{i=1}^{P}\frac{2^{rel_{i}}-1}{\log_{2}(i+1)}$$
We <mark style="background: #FFF3A3A6;">compute</mark> the <mark style="background: #FFF3A3A6;">best possible DCG</mark> i.e., the <mark style="background: #FFF3A3A6;">DCG </mark>if we<mark style="background: #FFF3A3A6;"> had ranked all relevant documents perfectly</mark>.
$$IDCG@K = DGC@K$$
To<mark style="background: #FFF3A3A6;"> make results</mark> <mark style="background: #FFF3A3A6;">comparable</mark> across queries or systems, we <mark style="background: #FFF3A3A6;">divide</mark> the actual $DCG$ by the <mark style="background: #FFF3A3A6;">ideal</mark> $DCG$


#### Evaluation Campaigns
<mark style="background: #FFF3A3A6;">Organised competitions</mark>, research groups <mark style="background: #FFF3A3A6;">test </mark>their <mark style="background: #FFF3A3A6;">search</mark> or <mark style="background: #FFF3A3A6;">ranking algorithms</mark> on <mark style="background: #FFF3A3A6;">shared datasets</mark> used<mark style="background: #FFF3A3A6;"> shared evaluation metrics</mark>. To <mark style="background: #FFF3A3A6;">evaluate</mark> a <mark style="background: #FFF3A3A6;">search engine,</mark> one<mark style="background: #FFF3A3A6;"> must know </mark>which <mark style="background: #FFF3A3A6;">documents</mark> are <mark style="background: #FFF3A3A6;">relevant</mark> to <mark style="background: #FFF3A3A6;">each query</mark>. But when dealing with billions of documents, <mark style="background: #FFF3A3A6;">cannot manually label</mark> all of them. <mark style="background: #FFF3A3A6;">Use</mark> a <mark style="background: #FFF3A3A6;">shortcut</mark> called<mark style="background: #FFF3A3A6;"> pooling:</mark>
1. Each<mark style="background: #FFF3A3A6;"> search system</mark> <mark style="background: #FFF3A3A6;">runs</mark> a <mark style="background: #FFF3A3A6;">fixed set</mark> of <mark style="background: #FFF3A3A6;">queries</mark> called <mark style="background: #FFF3A3A6;">evaluation topics</mark>, <mark style="background: #FFF3A3A6;">each system</mark> <mark style="background: #FFF3A3A6;">returns</mark> the <mark style="background: #FFF3A3A6;">top</mark> $N$ <mark style="background: #FFF3A3A6;">results</mark> for <mark style="background: #FFF3A3A6;">each query</mark>. 
2. A<mark style="background: #FFF3A3A6;"> central authority</mark> <mark style="background: #FFF3A3A6;">takes</mark> the <mark style="background: #FFF3A3A6;">union</mark> of <mark style="background: #FFF3A3A6;">all documents</mark> <mark style="background: #FFF3A3A6;">returned during</mark> that <mark style="background: #FFF3A3A6;">search</mark>
3. The <mark style="background: #FFF3A3A6;">union</mark> is randomly<mark style="background: #FFF3A3A6;"> split </mark>and <mark style="background: #FFF3A3A6;">assigned</mark> for<mark style="background: #FFF3A3A6;"> labelling</mark>
4. Each <mark style="background: #FFF3A3A6;">participant </mark><mark style="background: #FFF3A3A6;">receives</mark> a<mark style="background: #FFF3A3A6;"> subset</mark> of the <mark style="background: #FFF3A3A6;">pool</mark> and l<mark style="background: #FFF3A3A6;">abels each document</mark> as r<mark style="background: #FFF3A3A6;">elevant</mark>, <mark style="background: #FFF3A3A6;">partially relevant</mark>, <mark style="background: #FFF3A3A6;">not relevant</mark>
  5<mark style="background: #FFF3A3A6;">. After labelling, all judgments are aggregated so that each document gets a final, consensus based relevance label.</mark>

