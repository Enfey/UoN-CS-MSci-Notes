# Dealing with biased data

## Data Quality
- Issue of **ecological validity** of data sets
	- Extent to which study findings/data generalises to real-world settings
- **Real** callers vs **Simulated** callers
- **Differences in speech**
	- Recruited ppts talk more, speak faster
	- Real users ask for more help, interrupt system more

## Data statements
- Mitigating bias by being transparent
- "A data statement is a **characterisation of a dataset** that provides context to allow developers"
- Documented, standardised characterisation used to address ethical issues
- Often includes:
	- **curation rationale**
	- **language variety**
	- **speaker demographics**
	- **annotator demographics** 
	- **speech situation** 
	- **text charateristics such as genre**
	- **any previous processing or sub-selection steps.** 
- Better understand how experimental results may generalise and what biases might be reflected in systems built on the software.
- Specific to NLP, focusing heavily on linguistic variety, speaker demographics, and dialectal nuances.
## Data Sheets
- Standardises documentation framework; a standard datasheet follows a structured questionnaire covering several stages of the data lifecycle
	- **Motivation** - reasons for creating the dataset and info on funding/creators
	- **Composition** - what data represents, and its internal structure, including any labels and known errors
	- **Collection process** - detail;s aon how the data was acquired, who was involved and the timeframe of the collection
	- **Preprocessing/Cleaning/Labelling** - documentation of any transformations applied to the raw data
	- **Uses** - recommended tasks for the dataset and any environments where it should **not** be used
	- **Distribution and Maintenance** - info on how dataset is licensed and how will be updated or retired over time. 

# Crowdsourcing
> Delegation to targeted, distributed workforce who can perform tasks, often virtually. Commonly employed to obtain labelled data to train AI systems. Invisibility creates ethical and epistemic risks, may overlook who produces data and under what conditions. 

## Quality of labelling/crowdsourcing
- Training data quality is inseparable from human judgement. Research performing structured content analysis shows that many ML papers fail to report:
	- Who labelled the data?
	- Whether labellers were experts or crowdworkers
	- Training examples in the paper
	- How labellers were compensated
- Indicates concern, given how crucial quality of training data is and that labelling decisions embed values, assumptions, and cultural norms into datasets; bias at the data level becomes bias in the model. 

## Labour Conditions and Ethical Concerns
- Crowd work often resembles call-center labour in low-income contexts and is characterised by:
	- Low or unstable pay
	- No labour protections such as unions, minimum wage, sick leave
	- Power asymmetries between workers and requesters
- Projects like Turkopticon attempt to counter this by allowing workers to rate employers and avoid exploitative requesters, making invisible labour more visible and reframing crowdwork as real labour, not merely as technical input. 
### Technical fixes
- **Fair work** proposed automatically enforcing minimum wages for crowdworkers, fair work used a custom script to add a duration input to the bottom of Amazon Mechnical Turk tasks, which requesters can review, approve durations are correct, and can be used to calculate fair and consistent pay and send bonuses when needed.
- **Rehumanised Crowdsourcing** allocates microtasks based on demographic factors and compensation to mitigate biases in the contributor sample and increase the hourly pay given to contributors.

## Algorithmic solutions and transparency
- Word embeddings learn meaning from patterns in large text corpora. Because human language reflects social biases, **embeddings also absorb and reproduce those biases**.
- Analogies are solved via vector arithmetic, yielding some biological or role-based gender relations, and yielding some historical and cultural stereotypes e.g., man is to woman as computer programmer is to homemaker. The model is not sexist by intention, it mirrors sexist patterns in the data it was trained on. 
- Embeddings are used in many downstream systems, thus bias in data becomes bias in AI behaviour and can reinforce inequality. 
- We need **debiasing algorithms** to deal with this bias
- Research proposes identifying a **gender** direction in embedding space. They then introduce two methods.
	1. **Hard Debiasing**
		Completely removes the gender pair associations for gender neutral words e.g., explicitly equalises words like *doctor* and *nurse* so they are equidistant from gendered terms. Strong removal of bias but may remove meaningful semantic information.
	2. **Soft Debiasing**
		Reduces gender bias while preserving as much of the original structure as possible
		Tries to balance fairness with model performance.
- Debiasing methods however, are arguably cosmetic. Even after debiasing, gender bias can still be detected in word relationships; gender neutral words still cluster together by gender e.g., nurse and receptionist cluster together, engineer, architect cluster together, bias can be recovered using indirect cues. 
- Bias is structural and not a surface feature, and language encodes historical power relations; technical fixes are simply not holistic enough to fully solve socially produced bias. 
