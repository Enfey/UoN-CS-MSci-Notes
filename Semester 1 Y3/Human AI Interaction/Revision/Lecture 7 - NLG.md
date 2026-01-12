Post intent matching/NLU, NLG transforms machine rep into natural lang; occurs in generic NLP pipeline. 


# NLG
Subfield, AI, comp linguistics, produce, nat lang, from structured/unstructured data/IR.

## NLG use cases
- **Informational** : weather reports, medical summaries, financial reports
- **Entertainment** - story gen, AI rp
- **Interaction** - text interface to websites
- **Assistive** - aid ppl with special comms needs
- **Bot journalism/propaganda**


## Template-based vs Dynamic NLG
1. **Templated-based**
	Use fixed sentence patterns w/slots called templates, plug slots. Gap filling = simplest, then rule-based, which are extended gap-filling, then grammatical functions(more complex, not dynamic).
2. **Dynamic NLG**
	Use extensive rules/ML, build sentences dynamically, flexible, high error rate potential.

## How NLG fits into NLP pipeline
Convert input, tokens, further to some IR, intent match, then generate. It is the prepare result stage, turn structured data into fluent response. 


## General architecture of template-based NLG system
**1. Document planner**
	***Content determination*** - decide, what to say(communicative goal, lvl of precision)
	***Document structuring*** - decide, what order, to say info, produce natural results.
		E.g., S1 = I went to the store, S2 = I bought some fresh fruits.
		Structure 1 places S1 first and S2 second
		Structure 2 vice versa, structure 1 more natural. 
**2. Microplanner**
	***Lexicalisation*** - Choose, which words, use, depend on genre, context, perception
		word appropriate? would sentence, multiple interpretations?
	***Referring expression generation*** - expressions, referring, entities
		Multiple types: proper-noun noum, definite noun phrase, spatial, temporal, must be unambiguous, easy to understand.
	***Aggregation*** - merge syntactic/conceptual constituent sentences
		e.g., Paris is warm. London is warm -> Paris and London are warm. 
**3. Surface realiser**
	***Syntactic realisation*** - convert sentence plan, concrete syntactic structure.
	***Morphological realisation*** - applies inflectional morphology (tense, number, person, gender), selecting correct verb forms, pluralisation
	***Orphographic realisation*** - Converts morphologically realised output, proper written text, capitalisation rules, punctuation spacing, formatting conventions(quotations, hyphenation)

## Statistical Language models
Before neural models, NLG, $n-gram$ probability models. Given sequence, tokens, predict next token. Assign probability, sequence, words, decide, what sound natural, in lang. 

$P(\text{Nature is healing})$
$P(\text{Nature is killing})$

Good lang model assign $$P(\text{Nature is healing}) > P(\text{Nature is killing})$$

Lang models built on statistics from word counts. Probability of whole sentence. 
$P(w_1, w_2, w_3,...w_n) = P(w_1) \times P(w_2∣w_1) \times P(w_3∣w_1, w_2) \times \dots \times P$ 
Impossible to compute directly. Aka:
$$\prod_iP(w_i ∣w_1, w_2 \dots w_{i-1})$$
Simplify, each word, depends only on last few words.
$P(w_i∣w_{i-n+1}, \dots w_{i-1})$ 
Each word $i$ depends only on restricted set, preceding words. 

![[Pasted image 20251105212759.png]]
PICK THE ONE YOUR'RE ANALYSING, NOT THE WINDOW.

