# Algorithmic bias
Systematic+unfair discrim in outcomes produced by algorithms, particularly machine learning/AI driven algos. Occurs when algos consistently favour/disfavour certain groups/individuals.

## Why this occurs
- **Biased data**
	- Learn patterns from historical data, if data reflects social/historical inequality, algorithm perpetuates.
- **Biased design**
	- Developer may introduce bias via choices in model arch, training objectives, feature selection.
- **Feedback loops**
	- Algorithmic outputs can reinforce themselves via external consequences. Predictive policing, send more police to 'high-crime' areas, more arrests, reinforce label.
No inherent neutrality; reflect equity and biases of environment conceived in. 


# Racial bias
Prominent/harmful form of algorithmic bias; harm to individuals/groups based on ethnicity or race.

## Examples
- **Predictive policing**
	- COMPAS algorithms used in U.S. court to predict recidivism rate, predicted black defendants 'high-risk' 2x more than white counterparts.
- **Healthcare algorithms**
	- Study, found healthcare algo used to allocate extra care resources systematically underestimated needs of black patients as used **healthcare spending** as predictor for **health need**. Systemic inequality, extended to economic inequality, etc, lead algo to believe did not require as many resources as people from other races for same issues.
- **Hiring algorithms**
	- Amazon 2018 learned to prefer male applications, penalised resumes containing words like 'women's'.

# Bias in Facial recognition
Notorious, racial +gender bias
Study found commerical facial recognition systems had error rates under 1% for light-skinned men, and up to 35% for dark-skinned women. 

Misidentification by law enforcement is consequence of this bias, several wrong arrests traced to this. Lack of accountability too. 

# Targeted advertising + bias
Ad algos often reflect + amplify societal bias
- **Google ad system** - showed fewer high paying job ads to women
- **Facebook ads** - allowed advertisers to exclude users by race from seeing house, job, credit ads
- **Credit scoring** - produce biased results if infer sensitive traits indirectly through proxies like postcode + browsing behaviour.
Ad algos, optimise, engagement+click through, lead to, discriminatory targeting, if user groups behave differently online
Reinforcement bias, occur, when certain groups shown fewer/more lucrative activities e.g., gambling ads to poor postcode regions.


# Chatbots + Language models
Chatbots + gen AI systems, learn from giant copora, include racist, sexist, harmful content, can propagate discrimination.

**Examples**
- **Tay chatbot** - began posting racist + misogynistic tweets after being exposed to toxic online users
- **LLMs** can unconciously (often via jailbreaking) reproduce, stereotypes.
Risk of harms: encountering derogatory language, experience discrimination, warrant transparency about model limitations. 

## How to build chatbots around sensitive topics
Instead of blacklist, should handle sensitive topics e.g., race carefully. Do not assume can be avoided, anticipate it, and explicitly collect and aggregate dialogues to train in respectful conversations. Ensure diverse representation of cultural references inter-ethnically - diversity in devs more likely to lead to diverse database.

**NLP systems**, focus, probabilistic lang, doesn't tell you whether statement is problematic, need **meaning-in-context**.

## Word embeddings and bias
Numerical representations of words in continuous vector space, models like Word2Vec represent words such that words with similar meaning are close together in that space.

Word embeddings arise from training on large text corpora, inherit biases + stereotypes.
![[Pasted image 20251013205807.png]]
Quant tools developed to measure embedding bias such as **WEAT**, measures association strengths between target words and attribute words e.g., "male words" -> "career", "science".

Consequences of biased word embeddings, propagate, through various permutations of NLP systems.
- Search engines - biased autocomplete/ranking
- Sentiment analysis - e.g., black names -> negative sentiment
- Machine translation - translation gender neutral languages into gendered stereotypes.