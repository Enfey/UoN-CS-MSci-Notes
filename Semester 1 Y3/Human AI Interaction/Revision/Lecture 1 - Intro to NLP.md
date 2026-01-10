# NLU
- Branch of AI, subset of NLP, focuses on machine reading comprehension; understanding meaning, intent, context, in natural lang
- Underpins applications in reasoning, content analysis, QA, info extraction
- Breadth of NLU system determined by size/coverage of vocabulary, whereas depth is determined by how closely system's understanding approximates that of native speaker
- Techniques:
	- **Morphological analysis**
		- Understand word structure e.g., prefixes, suffixes, inflections
	- **Syntactic parsing**
		- Analysis of syntactic structure
	- **Discourse analysis**
		- Extracting meaning from input
	- **Knowledge grounding**
		- Linking lang to external models
	- **Pragmatic reasoning**
		- Intent recognition
- Ambiguity, context dependence, metaphor, common sense reasoning are ALL challenges.


# NLG
- Branch of AI, subset of NLP + comp linguistics, concerned with constructing systems that generate coherent + appropriate natural language from structured/unstructured data.
- Diasagreement, whether inputs to NLG system must be non-linguistic or linguistic (image descriptions vs paraphrasing)
- Complementtary to NLU
	- NLU maps natural lang to machine rep
	- NLG maps machine rep to natural lang
- NLG operates from known inputs, ambuiguity etc less central issues
- Stages:
	- **Content determination**
		- Selecting which info to express
	- **Document planning**
		- Structuring content into organised form
	- **Microplanning**
		- Deciding how to express content linguistically (lexical choice tc)
	- **Surface realisation**
		- Generating final grammatical text
	- **Post-processing**

# Historical trends in NLP
## Symbolic
- Early work, rule-based/knowledge driven systems, grounded in formal logic; handcrafted grammars, lexicons etc
- Parsing + generation via explicit rules
- Highly interpretable, scaling to open domains needed infeasible amounts of manual work, could not handle ambiguity+variability in real-world scenarios well.
## Statistical
- Began with availability of large corpora + better computational resources
- Language modelled probabilistically 
- Common techniques:
	- **n-gram language models** - predict probability of next word based on fixed window of preceding words
	- **Hidden markov models**
	- **Maximum entropy models** - used in POS tagging, entity recognition etc, models dependencies, does not assume conditional independence between consecutive states, consider past predictions when making current one
- Emphasise data-driven learning, evaluation metrics, scalable applications
- Struggles with long-range dependencies and deep semantic understanding

## Common NLP tasks
### Text classification
- Assigning predefined lables to text e.g., sentiment analysis, spam detection
### POS tagging
- Assign syntactic categories to words
### Entity recognition
- Identify+classify entities
### Syntactic parsing
- Deriving grammatical structure of sentences for use in downstream tasks e.g., relation extraction.
### Semantic role labelling
- Determine who did what to whom, when, where, assign roles to sentence constituents based on predicate-argument structure.
### Coreference resolution
- Determining which expressions refer to same entity.
### Machine translation

### QA
- Extractive(find answers in text), generative(produce full responses), open-domain(retrieve from large corpora)

## Prelims
- Each unit of interest we analyse = document
- Collection of documents is a corpus
- A set of consective words is an n-gram

### Preprocessing
![[Pasted image 20251013200455.png]]
#### Tokenisation
- Process of breaking text into smaller units = tokens. Similar to lexing in compilers w/o inclusion of lexical categorisation(though POS tagging often complementary). Facilitates more straightforward processing in downstream tasks.
#### POS tagging
Refers to category of words that share similar grammatical properties e.g., noun, verb, adjective, article, conjunction etc. Give each token its grammatical function, different languages have diff POS, different parser have diff POS tags for same lang. Help disambiguate meaning, using grammatical context, as reasoning vehicle.
### Word standardisation
#### Stemming
Recover stem of word, common part of all inflections of a given word. Walking and walked become walk, improve efficiency + accuracy, stem produced need not be identical to morphological root of word e.g., arguing becomes argu, acceptable as goal is just to group words, not be perfect.

Simple stemmer, look up inflected form in lookup table, but all inflected forms must be explicitly listed. 

Suffix stripping, rule-based, attempt to find root form by crudely removing suffix, also prefix stripping. Both = affix stripping. 

Stochastic algorithms, use probability to identify root form, learn  table of root form to inflected form relations. 

Either brute-force/exhaustive search or use implicit reasoning to stem word.

#### Lemmatisation
Process grouping inflected forms of a word to be analysed as single item, identified via word's lemma (morphological root), give more meaningful results. Trivial way to accomplish, via dictionary lookup. Need better system for cases such as in languages with long compound words. 

Process involves determining POS, applying different normalisation rules for each part of speech. Approach conditional upon obtaining correct POS.

### Stop word filtering
Cut out common words, not add much meaning, textual analysis, reducing noise in data, focus on meaningful characteristics that carry the semantic weight. Domain-dependent. 