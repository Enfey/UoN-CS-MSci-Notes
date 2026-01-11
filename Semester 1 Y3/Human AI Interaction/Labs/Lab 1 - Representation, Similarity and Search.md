One of most basic forms of artificial intelligence that people interact with on daily basis are search engines.
Lab objective - **build search engine.** 

## Prerequisites
`pip install numpy`
`pip install scipy`
Then, in python script import:
```python
import numpy
import scipy
```

Python has handy set of functions to save and load objects via  and a newer way of doings things with **joblib**, either works.
```python
from joblib import dump, load
dump(someobject, 'filename.joblib')
loaded_object = load('filename.joblib')
```
Important to know building an application that makes use of computationally processes, such as a machine learning model or an index, saves recomputation!

## Bag of Words
### Building a bag-of-words-model
Let us assume that documents are multi-sets of $n-grams$ . Let us take for example the first sentence of Mary Shelley's Frankenstein. 
$$\text{You will rejoice to hear that no disaster 
has accompanied the commencement of an enterprise which you have regarded with such evil forebodings.}$$

After tokenising and removing stop words, resulting set of unigram tokens is:
`[’you ’, ’rejoice ’, ’hear ’, ’disaster ’, ’ accompanied ’, ’ commencement ’, ’enterprise ’, ’regarded ’, ’evil ’, ’ forebodings ’]`

After removing punctuation and stemming each term, the **BoW** representation would be a list of those tokens with their frequency in the document
![[Pasted image 20251014010712.png]]

Look at another sentence from same novel:
$$\text{Six years have passed since I resolved on my present undertaking. I can, even now, remember the hour from which I dedicated myself to this great enterprise.}$$ 
Can stem these tokens, add to bag of words model computed for first passage: 
![[Pasted image 20251014010838.png]]
See that this matrix is very sparse - but representing documents as vectors allows us to have a common basis on which to compare them. 
Words used to build **BoW** model = $V$ = $Vocabulary$

### Term weighting
**BoW** model extremely powerful, and barring deep learning methods, the standard way of analysing textual documents. Runs into issue encoding documents of different lengths with regard to the weightings of each vector. Longer documents are skewed with regard to their weighting in accordance with their size. Leads to error when computing the similarity between documents.

Term-weighting methods come into play here to squash weights.

#### Binary weighting
Each term $t$ is either present or absent from document $d$. Puts large and small documents on same footing. 
$weight(t,d) = \Bigg\{ 1 \ \text{if t} \in d, 0 \ otherwise$ 
#### Log-frequency weighting
Each term $t$ is weighted by a function of the raw frequency in the document $d$. This solves the main drawback of frequency weighting by log-scaling term frequencies, which means that even a very frequent term will never be more than a few times as important as a less frequent one.
$weight(t,d) = log(1+freq(t, d))$

#### TF-IDF 
Each term is weighted using its log frequency multiplied by the inverse document frequency of that term.
$weight(t, d, n, N) = log(1+freq(t,d)) \times log(\frac{N}{n}$ )
	Where:
	 $N$ = number of documents
	 $n$ = number of documents containing that term
	 $d$ = the document
	 $t$ = term
The inverse document frequency is a handy way to ensure that a term that is present in almost every single document will have a low weight.

## Pre-processing data
```python
from urllib import request

doc_urls = {
	"Russia": "http://www.gutenberg.org/cache/epub/13437/pg13437.txt",
	"France": 
"http :// www.gutenberg.org /cache/epub/10577/ pg10577.txt ", 
	"England": 
"http://www.gutenberg.org/cache/epub/10135/pg10135 .txt",
	"USA": "http://www.gutenberg.org/cache/epub/10947/pg10947.txt",
	"Spain": "http://www.gutenberg.org/cache/epub/9987/pg9987.txt",
	"Scandinavia": "http://www.gutenberg.org/cache/epub/5336/pg5336.txt ",
	"Iceland":
"http://www.gutenberg.org/cache/epub/5603/pg5603.txt"
}

documents = {}

for country in doc_urls.keys():
	content = request.urlopen(doc_urls.get(country)).read.decode('utf8', errors='ignore')
	documents[country] = content
```

Scikit-Learn gives us access to its own functions for tokenising, filtering etc. Could work around them using the NLTK functions as they achieve the same things, but Sci-Kit Learn is easier to work with often.

Simplest function `CountVectorizer` which takes care of tokenising, filtering, and counting terms in documents.


```python
from urllib import request
from nltk.corpus import stopwords
from sklearn.feature_extraction.text import CountVectorizer

doc_urls = {
	"Russia": "http://www.gutenberg.org/cache/epub/13437/pg13437.txt",
	"France": 
"http :// www.gutenberg.org /cache/epub/10577/ pg10577.txt ", 
	"England": 
"http://www.gutenberg.org/cache/epub/10135/pg10135 .txt",
	"USA": "http://www.gutenberg.org/cache/epub/10947/pg10947.txt",
	"Spain": "http://www.gutenberg.org/cache/epub/9987/pg9987.txt",
	"Scandinavia": "http://www.gutenberg.org/cache/epub/5336/pg5336.txt ",
	"Iceland":
"http://www.gutenberg.org/cache/epub/5603/pg5603.txt"
}

documents = {}

for country in doc_urls.keys():
	content = request.urlopen(doc_urls.get(country)).read.decode('utf8', errors='ignore')
	documents[country] = content
	
all_text = documents.values()
count_vect = CountVectorizer(stopwords = stopwords.words('english'))

X_train_counts = count_vect.fit.transform(all_text)

print(X_train_counts.shape)
```
Returns a term-document matrix. Returns `(7, 24128)`. 24128 terms over 7 documents, approx 24k dimensional space.

The `CountVectorizer` can be instantiated with parameters to modify the vocabulary:
- `max_df`: maximum document frequency. Words which appear in more than `max_df` documents will not be considered. Can also provide floating point between 0 and 1, in which case filters based on percentage rather than raw number.
- `min_df`: minimum document frequency. Words which appear in fewer than `min_df` documents will not be considered. Can also provide float here.
- `max_features`: maximum number of terms in the vocabulary (cuts words with lowest frequency)
- `vocabulary`: if want to pre-specify the vocabulary want to use
- `binary`: treats frequency of terms in term-document matrix as binary.
If wanted to remove the top 10% of documents in terms of document frequency as well as all terms present in fewer than 5 documents, could do so by using the following command: 

`count_vect = CountVectorizer (stop_words = stopwords.words ('english'), min_df =5, max_df =0.9)`

Similarly, if you wanted to only keep the 1,000 most common unigrams:
`count_vect = CountVectorizer ( stop_words = stopwords.words (’english’), max_features=1000)`

The `ngram_range` parameter lets you change the tokenisation in order to analyse combinations of ngrams. A range of `(1,1)` would be unigrams only, while a range of `(2,2)` would be bigrams only.  Higher degree = more expensive to compute and stored.

For example, the following command would yield a vocabulary of size 737,043:
`count_vect = CountVectorizer (stop_words = stopwords.words (’english’), ngram_range =(1 ,4))`

### Term Weighting Methods
The `TfidfTransformer` method provided by Scikit-Learn, allows us apply transformation to simple counts of `CountVectorizer`. Has parameters:
- `use_idf` - can be `True` or `False`, specifies whether term weights will be transformed using their IDF weight. (usually set true)
- `smooth_idf` - can be true or false, specifies whether IDF weights will be smoothed.
- `sublinear_tf` - can be `True` or `False`, specifies whether term frequency is log scaled. (usually set true)
```python
from sklearn.feature_extraction.text import TfidfTransformer

count_vect = CountVectorizer(stop_words=stopwords.words('english'))
X_train_counts = count_vect.fit_transform(all_text)
tf_transformer = TfidfTransformer(use_idf = true, sublinear_tf=true).fit(X_train_counts)
X_train_tf = tf_transformer.transform(X_train_counts)
print(X_train_tf.shape)
```
Have term document matrix that almost any machine learning algorithm can understand and process. 

Care about weighting terms, even though cosine similarity examines angle, we do not want word count to be a deciding factor in our similarity calculation, and not all words are equally important either. Less extreme version of stopword filtering. 

### CountVectorizer stemming/lemmatising
Has additional argument `analyzer` to bring own stemmer easily.]
- `analyzer = 'word'` default option, means features will be word n-grams
- `analyzer = 'char'` means features will be character n-grams
- `analyzer = char_wb` means features will be character n-grams respecting word boundaries(no whitespace.)
- any callable function: passing callable function will make it use it to extract a sequence of features from the input.

```python
from sklearn.feature_extraction.text import CountVectorizer
from nltk.stem.snowball import PorterStemmer

p_stemmer = PorterStemmer()
analyzer = CountVectorizer().build_analyzer()

def stemmed_words(doc):
	return(p_stemmer.stem(w) for w in analyzer(doc))
	
stem_vectorizer=CountVectorizer(analyzer=stemmed_words)
#etc
```

### Term-Document Matrix
#### Document-Term Matrix
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
Storing everything in such a matrix permits useful operations, such as computing similarity of any pair of documents; which can be used to cluster them, rank them, do any sort of comparison. 

#### The query
As far as a search engine is concerned, a query is just another document. As such, when a user enters a search query, it is simply mapped to the vector space that was induced from the vocabulary of the document collection and gets treated as a very short document. 

#### Building Inverted Matrix
Document-Term matrix is useful, but data structure underlying search engines is the term-document matrix. The term-document matrix is simply the transpose of the document-term matrix. 
$$
X =
\begin{bmatrix}
x_{11} & x_{12} & \cdots & x_{1D} \\
x_{21} & x_{22} & \cdots & x_{2D} \\
\vdots & \vdots & \ddots & \vdots \\
x_{V1} & x_{V2} & \cdots & x_{VD}
\end{bmatrix}
$$
Instead now ask what documents contain a word, rather than what words are in this document. 

What happens is that each term essentially becomes a key in a dictionary, and each term points to a list of documents(and possibly extra info) where it appears. This is known as an inverted index.
	**Vocabulary** - all unique terms that appear in the corpus, acts as fast lookup-mechanism
	**Term-Document Matrix**(postings list) - for every term in dictionary, there is an associated list of documents containing that term. Aka postings list, each entry = posting
	![[Pasted image 20251028220255.png]]
In simplest form, each posting just contains document id, to support sophisticated ranking algorithms, postings often enriched with additional info. Can include term frequency, and positional information(where the word appears in the document)

##### Query Processing with an inverted index
Efficiency gain from inverted index becomes clear during query processing. To find documents matching multi-word query such as 'rejoice AND enterprise' system performs following steps:
1. Looks up term 'rejoic' in dictionary to retrieve its postings list
2. Looks up term 'enterpris' in dictionary to retrieve its postings list
3. Computes intersection of these two lists. Resulting list contains only the IDs of documents in both.
	Achievable in linear time, if postings lists sorted by document ID, traverse both lists simultaneously to find common docs via comparison.
```python
from collections import defaultdict #creates default val for new key

corpus = {
	'doc1' : [words!]
	'doc2' : [words]
	'doc3' : [words]
}

# lambda creates inner defaultdict when new term is seen, for that term
inverted_index = defaultdict(lambda: defaultdict(int))

for doc_id, tokens in corpus.items()
	for token in tokens:
		inverted_index[token][doc_id] += 1
		
final_index = {term: dict(postings) for term, postings in inverted_index.items()}

##e.g.,
{
  'cat': {'doc1': 1, 'doc3': 1},
  'sat': {'doc1': 1, 'doc2': 1},
  'dog': {'doc2': 1, 'doc3': 1},
  'on': {'doc1': 1, 'doc2': 1},
  'the': {'doc1': 1, 'doc2': 1},
  'mat': {'doc1': 1},
  'rug': {'doc2': 1},
  'chased': {'doc3': 1}
}
```
### Relevance and Similarity
Key function of an information retrieval system, computation of the **relevance** of a doc with respect to a query. Because query considered small doc, can approx. relevance via a **similarity** measure.

Goal of search engine, rank documents according to decreasing relevance, where relevance will be approximated as the **cosine similarity** between the query document and each document. 

#### Cosine Similarity
Cosine similarity between query document and a document $d$ is the cosine of the angle between the document vector and the query vector in the vector space induced by the vocabulary. It is defined as follows:
$$
\text{similarity}(q, d) = 
\frac{q \cdot d}{\|q\| \, \|d\|} =
\frac{\sum_{i=1}^{n} q_i d_i}{
\sqrt{\sum_{i=1}^{n} q_{i^2}} \, 
\sqrt{\sum_{i=1}^{n} d_{i^2}}}

$$
	$q_i$ and $d_i$ correspond to the $i^{th}$ component of the query and document vector. E.g., $q_2$ corresponds to frequency of term 2 in the query document. 
	$n =$ number of terms in document
Can implement by hand or use existing implementation. Scipy has cosine distance implemented. Numpy implementation below:

```python
from numpy import dot
from numpy.linalg import norm

query_doc  = [1, 2, 0, 0]
document_1 = [1, 5, 0, 0]

sim_1 = dot(query_doc, document_1) / (norm(query_doc)*norm(document_1))
```
### The Search
So now have relevance function via cosine similarity, an inverted index by way of term/document matrix, can build skeleton of a search engine. 
#### General algorithm
##### General algorithm for indexing:
1. Compute vocabulary of the corpus
2. Use the vocabulary to compute the term-document matrix of the corpus (alternatively transpose document-term matrix).
3. Apply term weighting to resulting term-document matrix.
##### General Algorithm for search:
1. Preprocess search query
2. Map search query onto document collection-induced vector space
3. Apply term weighting technique to tokens in the query
4. For each document in document collection:
	Compute cosine similarity with search query
5. Rank documents by decreasing similarity.

#### Parsing list of documents to build the index
```python
import os

document_path = "data"

corpus = {}
for file in os.listdir(document_path):
	filepath = document_path + os.sep + file
	with open(filepath, encoding='utf8', errors='ignore', mode = 'r') as document
		content = document.read()
		document_id = file
		corpus[document_id] = content
```
Stored all documents in dictionary. Use joblib to save this index and not recompute every time. 
#### Asking user for a query
```Python
stop = false

while not stop: 
	query = input('Enter query or STOP to quit:')
	
	if query == 'STOP':
		stop = True
	else:
		print(f'You are searching for{query}')
		#processing comes here
```

#### Confidence threshold and fallback mechanism
Key concept of human AI interaction is management of failure states. When searching for an answer to user query, always chance that query does not match anything in our corpus. Usually computed by some sort of arbitrary threshold in our similarity under which we consider that nothing is relevant. Few options:
##### Fallback by warning message
Simplest way to recover from state like this, display to user that there is nothing relevant to query, suggest reformulation. 
##### Fallback by query expansion
Automatically suggest query reformulation via process called **query expansion**. Many ways to do this. 

One may use a language model e.g., given existing query, what is most likely next term entered by user, word embedding representation would help. 

Could also use some sort of semantic resource such as dictionary or an ontology to refine the query.

Finally, something to keep in mind is that whatever adding to query should not take over from what the user entered. Good practice to apply some weighting function to make sure original terms are never complete overridden by expanded terms.

### Retrieval and Question Answering
Information retrieval offers great framework for question answering systems. Given a pre-existing corpus of Question-Answer pairs, a retrieval system would only need match a new question to existing questions in order to retrieve the most likely correct answer.

Designing a system in this way raises several interesting questions. How does one know whether the right question-answer pair has been matched? How is user input pre-processed as to not destroy its semantics? How can the system learn and improve? 
![[Pasted image 20251105084811.png]]
Several design decisions such as how to handle failure states, how to handle uncertainty, how to preprocess and index the data, are designed on a case by case basis by implementer.
### Evaluating Search Results
How does one know if the search algorithm is good? There are several metrics used in information retrieval that give an idea of whether a search engine is successful. 
We will use three of these: **Precision. Recall, Normalised Discounted Cumulative Gain(NDCG)**

#### Human Assessment of relevance
We measure the quality of a search engine based on a set of reference queries for which we know the expected results. The results can either be binary, or graded. In the former, relevance is encoded as 0 or 1, in the latter case relevance is encoded as a number from 0 to 2. 

A sample query would be represented as $$q_1 = \{d_{1} : 2, d_{5} : 2, d_{6}, : 2, d_{7}, d_{8} : 1\}$$ with the implicit assumption that it is 0 for all other documents. Knowing this, can then compute several metrics

#### Precision
Precision $@$ $K$ is concerned with how many of the returned $K$ documents are relevant. Focuses on only the top $K$ results, for example the first page of search results, as don't want this to be polluted with irrelevant documents. Usually evaluated with binary judgements, with no grading of relevance
$$Precision = \frac{|\{relevant documents\} \cap \{retrieved documents\}| }{|\{retrieved documents\}|}$$
E.g., 5 documents returned, 3 are relevant and retrieved, precision = $\frac{3}{5} = 0.6$
If we only wanted to consider the top 3 results, would filter, reduce both as necessary and recalculate. Tells how accurate retrieved results are.


#### Recall
Recall $@$ $K$ is concerned with how many of the relevant documents have been retrieved. Like precision, it is common to plot recall with multiple $K$ values to see how the results of algorithms change as more results are returned. Makes sense in small, specialised search engines where only a restricted set of documents in deemed relevant and coverage is considered most important. 

$$\frac{|\{relevant \ documents\} \cap \{retrieved \ documents\}|} {|\{relevant \ documents\}|}$$
Relevant across the whole collection, regardless of whether retrieved. Tells how complete results are. 


#### Normalised Discounted Cumulative Gain(NDCG)
Normalised Discounted Cumulative Gain (NDCG) $@$ $p$ is a way to evalute search results in a more fine-grained way; precision and recall treat relevance as binary, this supports graded relevance. 

We sum up the relevance scores of the top $P$ results
$$CG@P=\sum_{i=1}^{P}rel_{i}$$
Then discount lower ranks logarithmically
$$DCG@P\sum_{i=1}^{P}\frac{2^{rel_{i}}-1}{\log_{2}(i+1)}$$
Designed to reward systems that place relevant documents earlier, and penalise systems that bury them lower in the ranking

We compute the best possible DCG i.e., the DCG if we had ranked all relevant documents perfectly.
$$IDCG@K = DGC@K$$
To make results comparable across queries or systems, we divide the actual $DCG$ by the ideal $DCG$
$$DGC@K = \frac{DGC@K}{IDCG@K}$$
#### Evaluation Campaigns
Organised competitions, research groups test their search or ranking algorithms on shared datasets used shared evaluation metrics. To evaluate a search engine, one must know which documents are relevant to each query. But when dealing with billions of documents, cannot manually label all of them. Use a shortcut called pooling:
1. Each search system runs a fixed set of queries called evaluation topics, each system returns the top $N$ results for each query. 
2. A central authority takes the union of all documents returned during that search
3. The union is randomly split and assigned for labelling
4. Each participant receives a subset of the pool and labels each document as relevant, partially relevant, not relevant
5. After labelling, all judgments are aggregated so that each document gets a final, consensus based relevance label.

### Advanced Models
Bag of words and such are powerful, but have a fundamental limitation: they disregard word order and context. For these models, the sentences "The user queried the search engine" and "The search engine queried the user" are represented identically, as they contain the same set of words. The inability to capture syntax and semantics is worrying. Modern deep learning models aim to overcome this.

#### BERT
Stands for Bidirectional Encoder Representations from Transformers, processes text in a way that is fundamentally different from classic BoW methods. Reads the entire sequence on words in one step, this bidirectional context allow it to generate a unique representation for a word based on the words that appear both before and after it. For example, BERT would generate different vectors for the word "Bank" in the phrases "the river bank" and "the savings bank". The output of a BERT model is a dense vector, often called an embedding, of a fixed size. 