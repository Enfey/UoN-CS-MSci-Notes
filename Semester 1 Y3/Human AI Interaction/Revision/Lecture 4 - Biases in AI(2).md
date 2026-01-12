# Dealing with biased data
## Data quality
- **<mark style="background: #FFF3A3A6;">Ecological validity</mark>**
	- Extent, study findings/data, generalise, real-world
- **<mark style="background: #FFF3A3A6;">Real callers vs Simulated callers</mark>**
- **<mark style="background: #FFF3A3A6;">Differences in speech</mark>**
	- <mark style="background: #FFF3A3A6;">Recruited</mark> ppts <mark style="background: #FFF3A3A6;">talk more</mark>, <mark style="background: #FFF3A3A6;">speak faster</mark>
	- <mark style="background: #FFF3A3A6;">Real world</mark>, <mark style="background: #FFF3A3A6;">ask more help</mark>, <mark style="background: #FFF3A3A6;">interrupt more</mark>

## Data statements
- <mark style="background: #FFF3A3A6;">Mitigate bias</mark>, being <mark style="background: #FFF3A3A6;">transparent</mark>
- **Data statement** =<mark style="background: #FFF3A3A6;"> **characterisation** of dataset</mark>
- <mark style="background: #FFF3A3A6;">Documented</mark>, <mark style="background: #FFF3A3A6;">standardised</mark> characterisation, used,<mark style="background: #FFF3A3A6;"> address ethical issues.</mark>
- Includes:
	- **<mark style="background: #FFF3A3A6;">Curation rationale</mark>**
	- **<mark style="background: #FFF3A3A6;">Language variety</mark>**
	- **<mark style="background: #FFF3A3A6;">Speaker demographics</mark>**
	- **<mark style="background: #FFF3A3A6;">Annotator demographics</mark>**
	- **<mark style="background: #FFF3A3A6;">Speech situation</mark>**
	- **<mark style="background: #FFF3A3A6;">Text characteristics</mark> such as genre**
	- **<mark style="background: #FFF3A3A6;">Previous processing</mark>/sub-selection steps**
- Better <mark style="background: #FFF3A3A6;">understand</mark> how <mark style="background: #FFF3A3A6;">results </mark>may <mark style="background: #FFF3A3A6;">generalise</mark> + what <mark style="background: #FFF3A3A6;">biases </mark>may occur
-<mark style="background: #FFF3A3A6;"> Specific to NLP</mark>, focus heavily on <mark style="background: #FFF3A3A6;">linguistic variety</mark>

## Data sheets
- <mark style="background: #FFF3A3A6;">Standardised</mark>, <mark style="background: #FFF3A3A6;">documentation framework</mark>; <mark style="background: #FFF3A3A6;">structured questionnaire</mark>
	- <mark style="background: #FFF3A3A6;">Motivation </mark>- funding, creators, why
	- <mark style="background: #FFF3A3A6;">Composition </mark>- what data is, structure, labels, <mark style="background: #FFF3A3A6;">known errors</mark>
	- <mark style="background: #FFF3A3A6;">Collection process</mark> - incl w<mark style="background: #FFF3A3A6;">ho was involved</mark> + <mark style="background: #FFF3A3A6;">timeframe</mark>
	- <mark style="background: #FFF3A3A6;">Preprocessing</mark>/cleaning/labelling - <mark style="background: #FFF3A3A6;">docs of transformations</mark>
	- <mark style="background: #FFF3A3A6;">Uses </mark>-<mark style="background: #FFF3A3A6;"> recommended tasks</mark>
	- <mark style="background: #FFF3A3A6;">Distribution</mark> + <mark style="background: #FFF3A3A6;">maintenance</mark>

## Quality of labelling/crowdsourcing
- <mark style="background: #FFF3A3A6;">Training data quality</mark> = <mark style="background: #FFF3A3A6;">inseparable</mark> from <mark style="background: #FFF3A3A6;">human judgement</mark>
- <mark style="background: #FFF3A3A6;">Research performed content analysis</mark> shows <mark style="background: #FFF3A3A6;">many ML papers</mark> <mark style="background: #FFF3A3A6;">fail </mark>to<mark style="background: #FFF3A3A6;"> report:</mark>
	- <mark style="background: #FFF3A3A6;">Who labelled</mark> data
	- Whether labellers were <mark style="background: #FFF3A3A6;">experts</mark>
	- T<mark style="background: #FFF3A3A6;">raining examples</mark> 
	- <mark style="background: #FFF3A3A6;">How labellers compensated</mark>
- <mark style="background: #FFF3A3A6;">Concern</mark>; <mark style="background: #FFF3A3A6;">quality</mark> of <mark style="background: #FFF3A3A6;">training data</mark> = <mark style="background: #FFF3A3A6;">crucial;</mark> <mark style="background: #FFF3A3A6;">labelling decisions </mark><mark style="background: #FFF3A3A6;">embed values,</mark> <mark style="background: #FFF3A3A6;">assumptions</mark>, cultural norms. 

## Labour conditions and Ethical concerns
- <mark style="background: #FFF3A3A6;">Crowd work</mark>, resemble <mark style="background: #FFF3A3A6;">call center labour,</mark> characterised via: low pay, <mark style="background: #FFF3A3A6;">no labour protection</mark> ,<mark style="background: #FFF3A3A6;"> power asymmetries</mark>
- <mark style="background: #FFF3A3A6;">Projects counter</mark> by <mark style="background: #FFF3A3A6;">allowing workers</mark>,<mark style="background: #FFF3A3A6;"> rate employers</mark>, avoid exploitative jobs/requests.

### Technical fixes
- **<mark style="background: #FFF3A3A6;">Fair work</mark>** proposed <mark style="background: #FFF3A3A6;">automatically</mark> <mark style="background: #FFF3A3A6;">enforcing</mark> <mark style="background: #FFF3A3A6;">minimum wages</mark> for crowdworkers, fair work used custom script, <mark style="background: #FFF3A3A6;">add duration input</mark>, bottom, <mark style="background: #FFF3A3A6;">mechanical turk tasks</mark>, <mark style="background: #FFF3A3A6;">requesters can review</mark>, <mark style="background: #FFF3A3A6;">approve durations</mark>, <mark style="background: #FFF3A3A6;">calculate fair + consistent pay</mark> and send bonuses.
- **<mark style="background: #FFF3A3A6;">Rehumanised Crowdsourcing</mark>** <mark style="background: #FFF3A3A6;">allocates microtasks</mark><mark style="background: #FFF3A3A6;"> based on demographic factors</mark> + compensation, <mark style="background: #FFF3A3A6;">mitigate biases</mark> in contributor sample.

### Algorithmic solutions + transparencies
- <mark style="background: #FFF3A3A6;">Word embeddings,</mark> <mark style="background: #FFF3A3A6;">learn meaning</mark> from patterns in <mark style="background: #FFF3A3A6;">large corpora</mark>, <mark style="background: #FFF3A3A6;">lang reflects biases</mark>, embeddings <mark style="background: #FFF3A3A6;">absorb those biases</mark>
- <mark style="background: #FFF3A3A6;">Analogies solved</mark> via <mark style="background: #FFF3A3A6;">vector arithmetic</mark>, yield some biological or role-based gender relations etc, model not sexist by intention, <mark style="background: #FFF3A3A6;">simply symmetrical to training data</mark>
- <mark style="background: #FFF3A3A6;">Embeddings</mark> used in <mark style="background: #FFF3A3A6;">many downstream systems</mark>; <mark style="background: #FFF3A3A6;">bias in data </mark>= <mark style="background: #FFF3A3A6;">bias in behaviour</mark>
- Need **<mark style="background: #FFF3A3A6;">debiasing algorithms</mark>**
- Research, proposes, <mark style="background: #FFF3A3A6;">identifying gender direction</mark>, then <mark style="background: #FFF3A3A6;">introduce two methods</mark>. 
	1. **<mark style="background: #FFF3A3A6;">Hard debiasing</mark>**
		<mark style="background: #FFF3A3A6;">Completely remove</mark>, <mark style="background: #FFF3A3A6;">gender pair associations</mark> for g<mark style="background: #FFF3A3A6;">ender neutral words</mark> e.g., <mark style="background: #FFF3A3A6;">equalise words</mark> like<mark style="background: #FFF3A3A6;"> doctor + nurse</mark> so they are<mark style="background: #FFF3A3A6;"> equidistant</mark> from <mark style="background: #FFF3A3A6;">gendered terms.</mark> Remove all bias, but <mark style="background: #FFF3A3A6;">destroys semantics.</mark>
	2. **<mark style="background: #FFF3A3A6;">Soft debiasing</mark>**
		Reduce gender bias, while preserving as much of OG structure as possible. <mark style="background: #FFF3A3A6;">Balance fairness with model performance.</mark>
- **Debiasing**, <mark style="background: #FFF3A3A6;">arguably cosmetic</mark>, can <mark style="background: #FFF3A3A6;">still detect bias</mark> <mark style="background: #FFF3A3A6;">indirectly</mark> in word relationships <mark style="background: #FFF3A3A6;">e.g.,</mark> <mark style="background: #FFF3A3A6;">gender neutral words</mark> <mark style="background: #FFF3A3A6;">cluster together</mark> <mark style="background: #FFF3A3A6;">by gender</mark>.
- <mark style="background: #FFF3A3A6;">Bias = structural,</mark> <mark style="background: #FFF3A3A6;">not a surface feature</mark>,<mark style="background: #FFF3A3A6;"> technical fixes</mark> <mark style="background: #FFF3A3A6;">not holistic enough </mark>to solve socially produced bias.