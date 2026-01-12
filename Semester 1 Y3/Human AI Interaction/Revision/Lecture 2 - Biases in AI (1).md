# Algorithmic bias
<mark style="background: #FFF3A3A6;">Systematic+unfair discrim</mark> in <mark style="background: #FFF3A3A6;">outcomes </mark><mark style="background: #FFF3A3A6;">produced</mark> by <mark style="background: #FFF3A3A6;">algorithms,</mark> <mark style="background: #FFF3A3A6;">particularly machine learning/AI driven algos</mark>. Occurs when algos consistently favour/disfavour certain groups/individuals.

## Why this occurs
- **<mark style="background: #FFF3A3A6;">Biased data</mark>**
	- <mark style="background: #FFF3A3A6;">Learn patterns</mark> from <mark style="background: #FFF3A3A6;">historical data</mark>, if data reflects social/historical inequality, algorithm perpetuates.
- **<mark style="background: #FFF3A3A6;">Biased design</mark>**
	- <mark style="background: #FFF3A3A6;">Developer</mark> may<mark style="background: #FFF3A3A6;"> introduce bias</mark> via choices in <mark style="background: #FFF3A3A6;">model arch, training objectives, feature selection.</mark>
- **<mark style="background: #FFF3A3A6;">Feedback loops</mark>**
	- <mark style="background: #FFF3A3A6;">Algorithmic outputs</mark> can <mark style="background: #FFF3A3A6;">reinforce</mark> themselves <mark style="background: #FFF3A3A6;">via external consequences</mark>. <mark style="background: #FFF3A3A6;">Predictive policing</mark>, send more police to 'high-crime' areas, more arrests, reinforce label.
<mark style="background: #FFF3A3A6;">No inherent neutrality</mark>; reflect equity and biases of environment conceived in. 


# Racial bias
Prominent/harmful form of <mark style="background: #FFF3A3A6;">algorithmic</mark> bias; <mark style="background: #FFF3A3A6;">harm</mark> to individuals/groups <mark style="background: #FFF3A3A6;">based</mark> on <mark style="background: #FFF3A3A6;">ethnicity</mark> or <mark style="background: #FFF3A3A6;">race</mark>.

## Examples
- **Predictive policing**
	- <mark style="background: #FFF3A3A6;">COMPAS </mark>algorithms used in <mark style="background: #FFF3A3A6;">U.S. court</mark> to <mark style="background: #FFF3A3A6;">predict</mark> <mark style="background: #FFF3A3A6;">recidivism</mark> rate, <mark style="background: #FFF3A3A6;">predicted black defendants</mark> <mark style="background: #FFF3A3A6;">'high-risk'</mark> 2x <mark style="background: #FFF3A3A6;">more than white counterparts</mark>.
- **Healthcare algorithms**
	- <mark style="background: #FFF3A3A6;">Study</mark>, found <mark style="background: #FFF3A3A6;">healthcare algo</mark> <mark style="background: #FFF3A3A6;">used</mark> to <mark style="background: #FFF3A3A6;">allocate extra care resources</mark> systematically<mark style="background: #FFF3A3A6;"> underestimated</mark> <mark style="background: #FFF3A3A6;">needs</mark> of <mark style="background: #FFF3A3A6;">black patients </mark>as used <mark style="background: #FFF3A3A6;">**healthcare spending**</mark> as <mark style="background: #FFF3A3A6;">predictor</mark> for <mark style="background: #FFF3A3A6;">**health need**.</mark> <mark style="background: #FFF3A3A6;">Systemic inequality</mark>,<mark style="background: #FFF3A3A6;"> extended</mark> to <mark style="background: #FFF3A3A6;">economic inequality, </mark>etc, lead algo to believe did not require as many resources as people from other races for same issues.
- **Hiring algorithms**
	- <mark style="background: #FFF3A3A6;">Amazon 2018</mark> learned to <mark style="background: #FFF3A3A6;">prefer male applications</mark>,<mark style="background: #FFF3A3A6;"> penalised</mark> resumes containing <mark style="background: #FFF3A3A6;">words like 'women's'</mark>.

# Bias in Facial recognition
<mark style="background: #FFF3A3A6;">Notorious,</mark> racial +gender bias
Study found<mark style="background: #FFF3A3A6;"> commerical</mark><mark style="background: #FFF3A3A6;"> facial recognition systems</mark> had error rates <mark style="background: #FFF3A3A6;">under 1%</mark> for <mark style="background: #FFF3A3A6;">light-skinned</mark> men, and up to <mark style="background: #FFF3A3A6;">35%</mark> for d<mark style="background: #FFF3A3A6;">ark-skinned women. 
</mark>
<mark style="background: #FFF3A3A6;">Misidentification</mark> by <mark style="background: #FFF3A3A6;">law enforcement</mark> is <mark style="background: #FFF3A3A6;">consequence of this bias</mark>, several wrong arrests traced to this. Lack of accountability too. 

# Targeted advertising + bias
<mark style="background: #FFF3A3A6;">Ad algos often reflect + amplify societal bias</mark>
- **Google ad system** - <mark style="background: #FFF3A3A6;">showed fewer high paying job ads to women</mark>
- **Facebook ads** - <mark style="background: #FFF3A3A6;">allowed advertisers</mark> to <mark style="background: #FFF3A3A6;">exclude users by race</mark> from <mark style="background: #FFF3A3A6;">seeing house, job, credit ads</mark>
- **Credit scoring** - <mark style="background: #FFF3A3A6;">produce biased results </mark>if <mark style="background: #FFF3A3A6;">infer sensitive traits </mark>indirectly <mark style="background: #FFF3A3A6;">through proxies</mark> like <mark style="background: #FFF3A3A6;">postcode</mark> + <mark style="background: #FFF3A3A6;">browsing behaviour.</mark>
<mark style="background: #FFF3A3A6;">Ad algos, optimise, engagement+click through, lead to, discriminatory targeting,</mark> if user groups behave differently online
Reinforcement bias, occur, when certain groups shown fewer/more lucrative activities e.g., gambling ads to poor postcode regions.


# Chatbots + Language models
Chatbots + gen AI systems, <mark style="background: #FFF3A3A6;">learn from giant copora</mark>, <mark style="background: #FFF3A3A6;">include racist, sexist, harmful content</mark>, can <mark style="background: #FFF3A3A6;">propagate discrimination.</mark>

**Examples**
- **<mark style="background: #FFF3A3A6;">Tay chatbot</mark>** - began posting racist + misogynistic tweets after being exposed to toxic online users
- **<mark style="background: #FFF3A3A6;">LLMs</mark>** can unconciously (often via jailbreaking) reproduce, stereotypes.
Risk of harms: encountering derogatory language, experience discrimination, warrant transparency about model limitations. 

## How to build chatbots around sensitive topics
<mark style="background: #FFF3A3A6;">Instead of blacklist</mark>, should <mark style="background: #FFF3A3A6;">handle sensitive topics</mark> e.g., race <mark style="background: #FFF3A3A6;">carefully</mark>. <mark style="background: #FFF3A3A6;">Do not assume can be avoided, </mark>anticipate it, and explicitly collect and aggregate dialogues to train in respectful conversations. <mark style="background: #FFF3A3A6;">Ensure diverse representation of cultural references inter-ethnicall</mark>y - <mark style="background: #FFF3A3A6;">diversity in devs </mark>more likely to lead to <mark style="background: #FFF3A3A6;">diverse database.</mark>

<mark style="background: #FFF3A3A6;">**NLP systems**</mark>, focus, <mark style="background: #FFF3A3A6;">probabilistic lang</mark>, <mark style="background: #FFF3A3A6;">doesn't tell you</mark> whether <mark style="background: #FFF3A3A6;">statement is problematic, </mark>need **meaning-in-context**.

## Word embeddings and bias
<mark style="background: #FFF3A3A6;">Numerical representations</mark> of words in <mark style="background: #FFF3A3A6;">continuous vector space,</mark> models like Word2Vec <mark style="background: #FFF3A3A6;">represent words</mark> such that <mark style="background: #FFF3A3A6;">words with similar meaning</mark> are <mark style="background: #FFF3A3A6;">close together</mark> in that space.

Word embeddings arise from training on large text corpora, inherit biases + stereotypes.
![[Pasted image 20251013205807.png]]
<mark style="background: #FFF3A3A6;">Quant tools</mark> <mark style="background: #FFF3A3A6;">developed</mark> to<mark style="background: #FFF3A3A6;"> measure</mark> <mark style="background: #FFF3A3A6;">embedding bias</mark> such as <mark style="background: #FFF3A3A6;">**WEAT**,</mark> <mark style="background: #FFF3A3A6;">measures association strengths</mark> between <mark style="background: #FFF3A3A6;">target words</mark> and <mark style="background: #FFF3A3A6;">attribute words</mark> e.g., "male words" -> "career", "science".

<mark style="background: #FFF3A3A6;">Consequences</mark> of <mark style="background: #FFF3A3A6;">biased word embeddings</mark>, <mark style="background: #FFF3A3A6;">propagate</mark>, through various permutations of NLP systems.
- <mark style="background: #FFF3A3A6;">Search engines</mark> - <mark style="background: #FFF3A3A6;">biased autocomplete</mark>/ranking
- <mark style="background: #FFF3A3A6;">Sentiment analysis </mark>- e.g., <mark style="background: #FFF3A3A6;">black names -> negative sentiment</mark>
- <mark style="background: #FFF3A3A6;">Machine translation </mark>- <mark style="background: #FFF3A3A6;">translation gender neutral languages into gendered stereotypes</mark>.