
## List of differing approaches
- Rule based - define explicit rules to determine how and when to aggregate information
- Statistical - use statistical measures or patterns derived from large datasets to decide on aggregation e.g., certain sentence structures may be more common and preferred during aggregation
- Template based - define templates where multiple pieces of information can be inserted into predefined slots in the template
- Machine learning - train models on examples of aggregated content, allowing the model to learn and predict how the new content should be aggregated
### Content determination

Can adapt content determination based on user profiles/feedback

Ontologies, in context of content determination = structured frameworks, represent knowledge, about specific domain, categorising entities + defining relationships between them. For example, in the medical domain, if discussing a particular disease, an ontology might suggest including symptoms, causes , and treatments.

Query-driven determination, NLG system tied to query system, content determination influenced by user query.

### Document structuring
Decide order of info to produce more natural results.

#### Rules, heuristics 
Use predefined rules/schema + domain specific heuristics to dictate information presentation based on nature + relationship of content e.g., research paper start with intro, methods, results, discussion.

#### ML
Use ML, train models on structured docs to learn typical patterns + sequences for this kind of data

#### Discourse models
Make use, models, understand conventions, textual coherence, ensure document, follows recognised patterns, e.g., introduce topic, before going into details.


#### User guided structuring
Allow users, define/tweak structure according to preferences, make customisable. Make sense, if have reason, think,users, large difference, in how consume, info provided. 


#### Graph--based
Content = nodes, edges indicate relationships, algos traverse graph, determine optimal structure.

### Aggregation
Combine sep pieces info into coherent sensible concise sentences.

#### Syntactic aggregation
Merge info based on syntactic structures of sentences, without considering deeper semantics, perform on common subject+verb, and use conjunction to bridge.

#### Conceptual Aggregation
Merges info based on semantics of sentence, can reformulate sentences, convey combined concepts more effectively.

For example:
	John is a software developer
	John creates mobile applications.
Would yield the following contextual aggregation "John is a software developer specialising in mobile applications."
### Lexical choice
Appropriate words/phrases chosen, convey specific meaning, sentiment, style, tone; select best words to use in sentences given context+comms goal.

Factors influencing:
- **Specificity**
	- Some info, requires, more specific term, others can be more generic e.g., Canine vs Dog
- **Formality** - purchase vs buy
- **Tone and Sentiment**
- **Variability** - avoid repetition, choose equiv
- **Audience understanding** - consideration of audience familarity with terms is vital
#### Rule based
Predef rules/templates, dictate, which terms/phrases used , based on specific conditions.
#### Thesaurus based
Use thesaurus, find equiv terms, select them contextually.

#### Statistical methods
Statistical language models, determine likelihood/appropriateness of word/phrase in particular context.

#### ML
Deep learning, trained, vast amounts of text, make contextually appropriate lexical choices. 

#### User feedback
Incorporate user feedback; useful to learn more and personalise for user. 
### Referring expression generation
Creation of expressions, referring to entities. Focuses, determing how, refer to entities, such that, can be umambiguously recognised by reader. When generating nat lang, must refer entities, so audience can identify.

E.g., there are two apples on table, instead of just saying "The apple", a proper referring expression would specify "The green apple" or "The red apple".

Techniques+Considerations:
- **Attribute selection**
	- Determine which attributes essential to distinguish entity from others in its context
- **Definiteness**
	- Decide whether to use definite ("the cat") or indefinite ("a cat") based on familiarty/uniqueness of the entity
- **Pronomial reference**
	- Decide whether appropriate, use pronoun, based on prior metnions/clarity
- **Salience**
	- Entities that have been mentioned/play signficant role, more noticeable/salient, may refer to differently.
### Realisation
Final step, NLG process, structured IR, transformed, coherent, grammatically correct, fluent, nat lang

- **Morphological inflection/realisation**
	- Correct morphological forms of words, chosen, based on context
- **Punctuation/formatting(orthographic realisation)**
	- Proper punctuation marks, text correctly formatted.
- **Insertion of function words**
	- Words like 'of', 'is', may not be present in IR, inserted for grammatical correctness
Subtle overlap, prior stages, particularly aggregation in microplanning, have diff objectives; aggregation about content synthesis/eliminating redundancy, realisation about linguistic correctness  + fluency. Only a guideline, can happen in any order.



