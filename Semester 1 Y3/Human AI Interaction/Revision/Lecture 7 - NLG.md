<mark style="background: #FFF3A3A6;">Post intent matching/NLU</mark>, NLG <mark style="background: #FFF3A3A6;">transforms machine rep </mark>into <mark style="background: #FFF3A3A6;">natural lang</mark>; occurs in generic NLP pipeline. 


# NLG
<mark style="background: #FFF3A3A6;">Subfield</mark>, <mark style="background: #FFF3A3A6;">AI</mark>, <mark style="background: #FFF3A3A6;">comp linguistics</mark>, produce, <mark style="background: #FFF3A3A6;">nat lang</mark>, from <mark style="background: #FFF3A3A6;">structured</mark>/<mark style="background: #FFF3A3A6;">unstructured data</mark>/IR.

## NLG use cases
- **<mark style="background: #FFF3A3A6;">Informational</mark>** : <mark style="background: #FFF3A3A6;">weather reports</mark>, <mark style="background: #FFF3A3A6;">medical summaries,</mark> financial reports
- **<mark style="background: #FFF3A3A6;">Entertainment</mark>** - <mark style="background: #FFF3A3A6;">story gen</mark>, AI rp
- **<mark style="background: #FFF3A3A6;">Interaction</mark>** -<mark style="background: #FFF3A3A6;"> text interface</mark> to websites
- **<mark style="background: #FFF3A3A6;">Assistive</mark>** - <mark style="background: #FFF3A3A6;">aid ppl </mark>with <mark style="background: #FFF3A3A6;">special comms</mark> <mark style="background: #FFF3A3A6;">needs</mark>
- **<mark style="background: #FFF3A3A6;">Bot journalism</mark>/propaganda**


## Template-based vs Dynamic NLG
1. **<mark style="background: #FFF3A3A6;">Template-based</mark>**
	Use <mark style="background: #FFF3A3A6;">fixed sentence patterns</mark> w/<mark style="background: #FFF3A3A6;">slots</mark> called templates, <mark style="background: #FFF3A3A6;">plug slots</mark>. <mark style="background: #FFF3A3A6;">Gap filling = simplest,</mark> then <mark style="background: #FFF3A3A6;">rule-based,</mark> which are <mark style="background: #FFF3A3A6;">extended gap-filling</mark>, then <mark style="background: #FFF3A3A6;">grammatical functions</mark>(<mark style="background: #FFF3A3A6;">more complex</mark>, not dynamic).
2. **Dynamic NLG**
	<mark style="background: #FFF3A3A6;">Use extensive rules/ML</mark>, <mark style="background: #FFF3A3A6;">build sentences dynamically</mark>, flexible, <mark style="background: #FFF3A3A6;">high error rate potential.</mark>

## How NLG fits into NLP pipeline
Convert input, tokens, further to some IR, intent match, then generate. It is the prepare result stage, turn structured data into fluent response. 


## General architecture of template-based NLG system
**1. <mark style="background: #FFF3A3A6;">Document planner</mark>**
	***<mark style="background: #FFF3A3A6;">Content determination</mark>*** - decide, what to say(<mark style="background: #FFF3A3A6;">communicative goal</mark>, <mark style="background: #FFF3A3A6;">lvl of precision</mark>)
	***<mark style="background: #FFF3A3A6;">Document structuring</mark>*** - decide, <mark style="background: #FFF3A3A6;">what order</mark>, to <mark style="background: #FFF3A3A6;">say info</mark>, <mark style="background: #FFF3A3A6;">produce natural results.</mark>
		E.g., S1 = I went to the store, S2 = I bought some fresh fruits.
		Structure 1 places S1 first and S2 second
		Structure 2 vice versa, structure 1 more natural. 
**2. <mark style="background: #FFF3A3A6;">Microplanner</mark>**
	***<mark style="background: #FFF3A3A6;">Lexicalisation</mark>*** - <mark style="background: #FFF3A3A6;">Choose</mark>, <mark style="background: #FFF3A3A6;">which words,</mark> <mark style="background: #FFF3A3A6;">use</mark>, depend on genre, context, perception
		<mark style="background: #FFF3A3A6;">word appropriate</mark>? w<mark style="background: #FFF3A3A6;">ould sentence, multiple interpretations</mark>?
	***<mark style="background: #FFF3A3A6;">Referring expression generation</mark>*** - <mark style="background: #FFF3A3A6;">expressions</mark>, <mark style="background: #FFF3A3A6;">referring, entities</mark>
		Multiple types: <mark style="background: #FFF3A3A6;">proper-noun noun</mark>,<mark style="background: #FFF3A3A6;"> definite noun phrase</mark>, <mark style="background: #FFF3A3A6;">spatial,</mark><mark style="background: #FFF3A3A6;"> temporal</mark>, must be <mark style="background: #FFF3A3A6;">unambiguous</mark>, <mark style="background: #FFF3A3A6;">easy to understand.</mark>
	***Aggregation*** -<mark style="background: #FFF3A3A6;"> merge syntactic/conceptual constituent sentences</mark>
		e.g., Paris is warm. London is warm -> Paris and London are warm. 
**3. Surface realiser**
	***<mark style="background: #FFF3A3A6;">Syntactic realisation</mark>*** - <mark style="background: #FFF3A3A6;">convert sentence plan</mark>, <mark style="background: #FFF3A3A6;">concrete syntactic structure</mark>.
	***<mark style="background: #FFF3A3A6;">Morphological realisation</mark>*** -<mark style="background: #FFF3A3A6;"> applies inflectional morphology</mark> (tense, number, person, gender),<mark style="background: #FFF3A3A6;"> selecting correct verb forms, </mark><mark style="background: #FFF3A3A6;">pluralisation</mark>
	***<mark style="background: #FFF3A3A6;">Orphographic realisation</mark>*** -<mark style="background: #FFF3A3A6;"> Converts morphologically realised output</mark>,<mark style="background: #FFF3A3A6;"> proper written text</mark>, <mark style="background: #FFF3A3A6;">capitalisation</mark> rules, <mark style="background: #FFF3A3A6;">punctuation</mark> spacing, formatting conventions(quotations, hyphenation)

## Statistical Language models
<mark style="background: #FFF3A3A6;">Before neural models</mark>, NLG, $n-gram$ <mark style="background: #FFF3A3A6;">probability models</mark><mark style="background: #FFF3A3A6;">. Given sequence</mark>, tokens, <mark style="background: #FFF3A3A6;">predict next token</mark>. <mark style="background: #FFF3A3A6;">Assign probability</mark>, <mark style="background: #FFF3A3A6;">sequence</mark>, words, <mark style="background: #FFF3A3A6;">decide</mark>, <mark style="background: #FFF3A3A6;">what sound natural</mark>, in lang. 

$P(\text{Nature is healing})$
$P(\text{Nature is killing})$

<mark style="background: #FFF3A3A6;">Good lang model assign </mark>$$P(\text{Nature is healing}) > P(\text{Nature is killing})$$

<mark style="background: #FFF3A3A6;">Lang models</mark> <mark style="background: #FFF3A3A6;">built</mark> on <mark style="background: #FFF3A3A6;">statistics from word counts</mark>. <mark style="background: #FFF3A3A6;">Probability</mark> of <mark style="background: #FFF3A3A6;">whole sentence. </mark>
$P(w_1, w_2, w_3,...w_n) = P(w_1) \times P(w_2∣w_1) \times P(w_3∣w_1, w_2) \times \dots \times P$ 
<mark style="background: #FFF3A3A6;">Impossible</mark> to <mark style="background: #FFF3A3A6;">compute directly. </mark>Aka:
$$\prod_iP(w_i ∣w_1, w_2 \dots w_{i-1})$$
<mark style="background: #FFF3A3A6;">Simplify</mark>, <mark style="background: #FFF3A3A6;">each word</mark>, <mark style="background: #FFF3A3A6;">depends only</mark> on <mark style="background: #FFF3A3A6;">last few words.</mark>
$P(w_i∣w_{i-n+1}, \dots w_{i-1})$ 
<mark style="background: #FFF3A3A6;">Each word</mark> $i$ <mark style="background: #FFF3A3A6;">depends only</mark> on<mark style="background: #FFF3A3A6;"> restricted set</mark>, preceding words. 

![[Pasted image 20251105212759.png]]
DIVIDE WHOLE TOKEN/CONTEXT BY JUST THE CONTEXT PART, RHS OF PROBABILITY SIMPLIFIED.

