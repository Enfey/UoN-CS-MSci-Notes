# Dealing with biased data
## Data quality
- **Ecological validity**
	- Extent, study findings/data, generalise, real-world
- **Real callers vs Simulated callers**
- **Differences in speech**
	- Recruited ppts talk more, speak faster
	- Real world, ask more help, interrupt more

## Data statements
- Mitigate bias, being transparent
- **Data statement** = **characterisation** of dataset
- Documented, standardised characterisation, used, address ethical issues.
- Includes:
	- **Curation rationale**
	- **Language variety**
	- **Speaker demographics**
	- **Annotator demographics**
	- **Speech situation**
	- **Text characteristics such as genre**
	- **Previous processing/sub-selection steps**
- Better understand how results may generalise + what biases may occur
- Specific to NLP, focus heavily on linguistic variety

## Data sheets
- Standardised, documentation framework; structured questionnaire
	- Motivation - funding, creators, why
	- Composition - what data is, structure, labels, known errors
	- Collection process - incl who was involved + timeframe
	- Preprocessing/cleaning/labelling - docs of transformations
	- Uses - recommended tasks
	- Distribution + maintenance

## Quality of labelling/crowdsourcing
- Training data quality = inseparable from human judgement
- Research performed content analysis shows many ML papers fail to report:
	- Who labelled data
	- Whether labellers were experts
	- Training examples 
	- How labellers compensated
- Concern; quality of training data = crucial; labelling decisions embed values, assumptions, cultural norms. 

## Labour conditions and Ethical concerns
- Crowd work, resemble call center labour, characterised via: low pay, no labour protection , power asymmetries
- Projects counter by allowing workers, rate employers, avoid exploitative jobs/requests.

### Technical fixes
- **Fair work** proposed automatically enforcing minimum wages for crowdworkers, fair work used custom script, add duration input, bottom, mechanical turk tasks, requesters can review, approve durations, calculate fair + consistent pay and send bonuses.
- **Rehumanised Crowdsourcing** allocates microtasks based on demographic factors + compensation, mitigate biases in contributor sample.

### Algorithmic solutions + transparencies
- Word embeddings, learn meaning from patterns in large corpora, lang reflects biases, embeddings absorb those biases
- Analogies solved via vector arithmetic, yield some biological or role-based gender relations etc, model not sexist by intention, simply symmetrical to training data
- Embeddings used in many downstream systems; bias in data = bias in behaviour
- Need **debiasing algorithms**
- Research, proposes, identifying gender direction, then introduce two methods. 
	1. **Hard debiasing**
		Completely remove, gender pair associations for gender neutral words e.g., equalise words like doctor + nurse so they are equidistant from gendered terms. Remove all bias, but destroys semantics.
	2. **Soft debiasing**
		Reduce gender bias, while preserving as much of OG structure as possible. Balance fairness with model performance.
- **Debiasing**, arguably cosmetic, can still detect bias in word relationships e.g., gender neutral words cluster together by gender, recover bias indirectly.
- Bias = structural, not a surface feature, technical fixes not holistic enough to solve socially produced bias.