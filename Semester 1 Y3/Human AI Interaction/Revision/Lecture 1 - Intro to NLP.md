# NLU
- Branch of AI, <mark style="background: #FFF3A3A6;">subset of NLP</mark>, focuses on <mark style="background: #FFF3A3A6;">machine reading comprehension</mark>; understanding <mark style="background: #FFF3A3A6;">meaning</mark>, <mark style="background: #FFF3A3A6;">intent</mark>,<mark style="background: #FFF3A3A6;"> context</mark>, in <mark style="background: #FFF3A3A6;">natural lang</mark>
- Underpins applications in <mark style="background: #FFF3A3A6;">reasoning</mark>, <mark style="background: #FFF3A3A6;">content analysis</mark>, <mark style="background: #FFF3A3A6;">QA</mark>, <mark style="background: #FFF3A3A6;">info extraction</mark>
- <mark style="background: #FFF3A3A6;">Breadth</mark> of NLU system <mark style="background: #FFF3A3A6;">determined</mark> by <mark style="background: #FFF3A3A6;">size/coverage</mark> of <mark style="background: #FFF3A3A6;">vocabulary,</mark> whereas <mark style="background: #FFF3A3A6;">depth</mark> is determined by <mark style="background: #FFF3A3A6;">how closely</mark> <mark style="background: #FFF3A3A6;">system's understanding</mark> approximates that of<mark style="background: #FFF3A3A6;"> native speaker</mark>
- Techniques:
	- **<mark style="background: #FFF3A3A6;">Morphological analysis</mark>**
		- Understand word structure e.g., prefixes, suffixes, inflections
	- **<mark style="background: #FFF3A3A6;">Syntactic parsing</mark>**
		- Analysis of syntactic structure
	- <mark style="background: #FFF3A3A6;">**Discourse analysis**</mark>
		- Extracting meaning from input
	- **<mark style="background: #FFF3A3A6;">Knowledge grounding</mark>**
		- Linking lang to external models
	- **<mark style="background: #FFF3A3A6;">Pragmatic reasoning</mark>**
		- Intent recognition
- <mark style="background: #FFF3A3A6;">Ambiguity</mark>, <mark style="background: #FFF3A3A6;">context dependence</mark>, <mark style="background: #FFF3A3A6;">metaphor</mark>, <mark style="background: #FFF3A3A6;">common sense reasoning</mark> are ALL challenges.


# NLG
- Branch of AI, <mark style="background: #FFF3A3A6;">subset of NLP</mark> + <mark style="background: #FFF3A3A6;">comp linguistics</mark>, concerned with <mark style="background: #FFF3A3A6;">constructing systems</mark> that <mark style="background: #FFF3A3A6;">generate coherent</mark> + appropriate <mark style="background: #FFF3A3A6;">natural language</mark> from <mark style="background: #FFF3A3A6;">structured/unstructured</mark> data.
- Diasagreement, whether <mark style="background: #FFF3A3A6;">inputs</mark> to NLG system must be <mark style="background: #FFF3A3A6;">non-linguistic</mark> or<mark style="background: #FFF3A3A6;"> linguistic </mark>(image descriptions vs paraphrasing)
- Complementtary to NLU
	- <mark style="background: #FFF3A3A6;">NLU</mark> <mark style="background: #FFF3A3A6;">maps natural lang</mark> to <mark style="background: #FFF3A3A6;">machine rep</mark>
	- <mark style="background: #FFF3A3A6;">NLG</mark> <mark style="background: #FFF3A3A6;">maps machine rep</mark> to <mark style="background: #FFF3A3A6;">natural lang</mark>
- <mark style="background: #FFF3A3A6;">NLG</mark> operates from <mark style="background: #FFF3A3A6;">known inputs</mark>,<mark style="background: #FFF3A3A6;"> ambuiguity</mark> etc <mark style="background: #FFF3A3A6;">less central issues</mark>
- Stages:
	- **<mark style="background: #FFF3A3A6;">Content determination</mark>**
		- Selecting which info to express
	- **<mark style="background: #FFF3A3A6;">Document planning</mark>**
		- Structuring content into organised form
	- **<mark style="background: #FFF3A3A6;">Microplanning</mark>**
		- Deciding how to express content linguistically (lexical choice tc)
	- **<mark style="background: #FFF3A3A6;">Surface realisation</mark>**
		- Generating final grammatical text
	- **<mark style="background: #FFF3A3A6;">Post-processing</mark>**

# Historical trends in NLP
## Symbolic
- <mark style="background: #FFF3A3A6;">Early work</mark>, <mark style="background: #FFF3A3A6;">rule-based</mark>/knowledge driven systems,<mark style="background: #FFF3A3A6;"> grounded in formal logic</mark>; handcrafted grammars, lexicons etc
- Parsing + generation <mark style="background: #FFF3A3A6;">via explicit rules</mark>
- Highly<mark style="background: #FFF3A3A6;"> interpretable</mark>, <mark style="background: #FFF3A3A6;">scaling to open domains</mark> needed <mark style="background: #FFF3A3A6;">infeasible</mark> amounts of manual work, <mark style="background: #FFF3A3A6;">could not handle ambiguity+variability</mark> in real-world scenarios well.
## Statistical
- Began with availability of large corpora + better computational resources
- <mark style="background: #FFF3A3A6;">Language modelled probabilistically </mark>
- Common techniques:
	- <mark style="background: #FFF3A3A6;">**n-gram language models**</mark> - <mark style="background: #FFF3A3A6;">predict probability</mark> of next word <mark style="background: #FFF3A3A6;">based on fixed window</mark> of preceding words
	- <mark style="background: #FFF3A3A6;">**Hidden markov models**</mark>
	- <mark style="background: #FFF3A3A6;">**Maximum entropy models** </mark>- used in POS tagging, entity recognition etc, models dependencies, does not assume conditional independence between consecutive states, consider past predictions when making current one
- <mark style="background: #FFF3A3A6;">Emphasise data-driven learning</mark>, <mark style="background: #FFF3A3A6;">evaluation metrics</mark>, scalable applications
- <mark style="background: #FFF3A3A6;">Struggles with long-range dependencies </mark>and <mark style="background: #FFF3A3A6;">deep semantic understanding</mark>

## Common NLP tasks
### <mark style="background: #FFF3A3A6;">Text classification</mark>
- Assigning predefined lables to text e.g., sentiment analysis, spam detection
### <mark style="background: #FFF3A3A6;">POS tagging</mark>
- Assign syntactic categories to words
### <mark style="background: #FFF3A3A6;">Entity recognition</mark>
- Identify+classify entities
### <mark style="background: #FFF3A3A6;">Syntactic parsing</mark>
- Deriving grammatical structure of sentences for use in downstream tasks e.g., relation extraction.
### <mark style="background: #FFF3A3A6;">Semantic role labelling</mark>
-<mark style="background: #FFF3A3A6;"> Determine </mark><mark style="background: #FFF3A3A6;">who did what to whom</mark>, when, where, assign roles to sentence constituents based on predicate-argument structure.
### <mark style="background: #FFF3A3A6;">Coreference resolution</mark>
- Determining which expressions refer to same entity.
### <mark style="background: #FFF3A3A6;">Machine translation</mark>

### <mark style="background: #FFF3A3A6;">QA</mark>
- Extractive(find answers in text), generative(produce full responses), open-domain(retrieve from large corpora)

## <mark style="background: #FFF3A3A6;">Prelims</mark>
- Each unit of interest we analyse = document
- Collection of documents is a corpus
- A set of consective words is an n-gram

### Preprocessing
![[Pasted image 20251013200455.png]]
#### Tokenisation
- Process of <mark style="background: #FFF3A3A6;">breaking text</mark> into <mark style="background: #FFF3A3A6;">smaller units</mark> = tokens. Similar to lexing in compilers w/o inclusion of lexical categorisation(though POS tagging often complementary). <mark style="background: #FFF3A3A6;">Facilitates</mark> more <mark style="background: #FFF3A3A6;">straightforward processing</mark> in <mark style="background: #FFF3A3A6;">downstream tasks.</mark>
#### POS tagging
Refers to <mark style="background: #FFF3A3A6;">category of words</mark> that <mark style="background: #FFF3A3A6;">share similar grammatical properties </mark>e.g., noun, verb, adjective, article, conjunction etc. <mark style="background: #FFF3A3A6;">Give each token</mark> its <mark style="background: #FFF3A3A6;">grammatical function</mark>, different languages have diff POS, different parser have diff POS tags for same lang. <mark style="background: #FFF3A3A6;">Help disambiguate meaning</mark>, using <mark style="background: #FFF3A3A6;">grammatical context</mark>, as <mark style="background: #FFF3A3A6;">reasoning vehicle.</mark>
### Word standardisation
#### Stemming
<mark style="background: #FFF3A3A6;">Recover stem of word</mark>, <mark style="background: #FFF3A3A6;">common part</mark> of <mark style="background: #FFF3A3A6;">all inflections</mark> of a given word. Walking and walked become walk, improve efficiency + accuracy, <mark style="background: #FFF3A3A6;">stem produced need not be identical to morphological root</mark> of word e.g., arguing becomes argu, acceptable as goal is just to <mark style="background: #FFF3A3A6;">group words, not be perfect.
</mark>
<mark style="background: #FFF3A3A6;">Simple stemmer</mark>, <mark style="background: #FFF3A3A6;">look up inflected form</mark> in lookup table, but all inflected forms must be explicitly listed. 

<mark style="background: #FFF3A3A6;">Suffix stripping</mark>, rule-based, attempt to find root form by crudely removing suffix, also <mark style="background: #FFF3A3A6;">prefix stripping</mark>. Both = <mark style="background: #FFF3A3A6;">affix stripping.</mark> 

<mark style="background: #FFF3A3A6;">Stochastic algorithms</mark>, use <mark style="background: #FFF3A3A6;">probability</mark> to<mark style="background: #FFF3A3A6;"> identify root form</mark>, learn  table of root form to inflected form relations. 

<mark style="background: #FFF3A3A6;">Either brute-force/exhaustive search</mark> <mark style="background: #FFF3A3A6;">or use implicit reasoning</mark> to stem word.

#### Lemmatisation
<mark style="background: #FFF3A3A6;">Process grouping inflected forms of a word</mark> to <mark style="background: #FFF3A3A6;">be analysed as single item</mark>, <mark style="background: #FFF3A3A6;">identified via word's lemma</mark> (morphological root), <mark style="background: #FFF3A3A6;">give more meaningful results</mark>. Trivial way to accomplish, via <mark style="background: #FFF3A3A6;">dictionary lookup.</mark> Need <mark style="background: #FFF3A3A6;">better system for cases</mark> such as in <mark style="background: #FFF3A3A6;">languages</mark> with <mark style="background: #FFF3A3A6;">long compound words. </mark>

<mark style="background: #FFF3A3A6;">Process</mark> involves <mark style="background: #FFF3A3A6;">determining POS</mark>, <mark style="background: #FFF3A3A6;">applying different normalisation rules</mark> for each part of speech. <mark style="background: #FFF3A3A6;">Approach conditional </mark>upon o<mark style="background: #FFF3A3A6;">btaining correct POS.</mark>

### Stop word filtering
Cut out common words, not add much meaning, textual analysis, reducing noise in data, focus on meaningful characteristics that carry the semantic weight. Domain-dependent. 