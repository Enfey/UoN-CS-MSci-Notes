## Discoverability
><mark style="background: #FFF3A3A6;"> ease</mark>, which <mark style="background: #FFF3A3A6;">users</mark>, <mark style="background: #FFF3A3A6;">find</mark>, execute, system<mark style="background: #FFF3A3A6;"> commands</mark>/features

<mark style="background: #FFF3A3A6;">Challenging</mark>, <mark style="background: #FFF3A3A6;">VUI</mark> context, no visual cues

An **<mark style="background: #FFF3A3A6;">affordance</mark>** = <mark style="background: #FFF3A3A6;">action possibilities</mark> <mark style="background: #FFF3A3A6;">environment</mark> <mark style="background: #FFF3A3A6;">offers</mark>. Affordance has <mark style="background: #FFF3A3A6;">narrow meaning</mark>; **<mark style="background: #FFF3A3A6;">possible actions</mark> actor<mark style="background: #FFF3A3A6;"> can perceive</mark>**.


### Discoverability GUI vs VUI
- **<mark style="background: #FFF3A3A6;">Recognition</mark>**
- **<mark style="background: #FFF3A3A6;">Recall</mark>**
- <mark style="background: #FFF3A3A6;">Recognition better</mark>, <mark style="background: #FFF3A3A6;">minimise cognitive l,oad</mark>
- <mark style="background: #FFF3A3A6;">Voice</mark> interface,<mark style="background: #FFF3A3A6;"> principle violated</mark>

### Discoverability challenge
- Invisibility+<mark style="background: #FFF3A3A6;">ephmerality of speech</mark>, exacerbate issues, difficult users find + exec available functions
- Can <mark style="background: #FFF3A3A6;">hurt</mark>:
	- <mark style="background: #FFF3A3A6;">Task success</mark>
	- <mark style="background: #FFF3A3A6;">Efficiency</mark>
	- <mark style="background: #FFF3A3A6;">User satisfaction</mark>
- **Study task** - <mark style="background: #FFF3A3A6;">order 2 diff dishes</mark> from <mark style="background: #FFF3A3A6;">local takeaway</mark> 
	- **Study conditions**
		- <mark style="background: #FFF3A3A6;">Automatic help</mark>
		-<mark style="background: #FFF3A3A6;"> Requested help</mark>
		- <mark style="background: #FFF3A3A6;">No help(baseline)</mark>
	- Measured
		-<mark style="background: #FFF3A3A6;"> Perf metrics</mark>
			- <mark style="background: #FFF3A3A6;">Time per task</mark>
			- <mark style="background: #FFF3A3A6;">Turns per task</mark>
			- <mark style="background: #FFF3A3A6;">Errors per task</mark>
			- <mark style="background: #FFF3A3A6;">Completed stages</mark>
		- <mark style="background: #FFF3A3A6;">Perceived usability</mark>
			-<mark style="background: #FFF3A3A6;"> SUS</mark>
			- <mark style="background: #FFF3A3A6;">Post-task discussion feedback</mark>
	- <mark style="background: #FFF3A3A6;">Automatic help = best</mark>, performance in requested varied, baseline worst,<mark style="background: #FFF3A3A6;"> turns per task</mark> only metric for which <mark style="background: #FFF3A3A6;">sig diff </mark>found between requested and automatic.
### Discoverability support
- **Automatic help**
	- System proactively tell users what can say
- **Requested help**
	- Only give help when asked
- **No help**
	- No guidance unless designed into flow, must guess.


## Response design
- Gave echo to 5 households, conditional voice recorder, study real conversational breakdowns. 

### Findings
- <mark style="background: #FFF3A3A6;">Voice interaction</mark> =<mark style="background: #FFF3A3A6;"> messy</mark>, people <mark style="background: #FFF3A3A6;">repeat selves</mark>, change phrasing, talk over each other
- <mark style="background: #FFF3A3A6;">30-50%</mark> <mark style="background: #FFF3A3A6;">failure rates</mark> in everyday interactions
	- <mark style="background: #FFF3A3A6;">Misrecognition</mark>
	- <mark style="background: #FFF3A3A6;">Misinterpretation</mark>
	- <mark style="background: #FFF3A3A6;">Wrong domain</mark>
	- <mark style="background: #FFF3A3A6;">Ambiguous intent</mark>
- Need <mark style="background: #FFF3A3A6;">good error recovery</mark>, explain problem, offer steps.

### Recovery, repair, progressivity
- **Recovery**
	- <mark style="background: #FFF3A3A6;">Recover from error</mark>, <mark style="background: #FFF3A3A6;">say what went wrong</mark> e.g., instead of "I didn't understand" a prompt like "I couldn't find a quiz called 'family quiz'"
- **Repair**
	- <mark style="background: #FFF3A3A6;">Help user fix their request</mark>, give resolution "You can say start a quiz", teach the system's language
- **Progressivity**
	- Help **<mark style="background: #FFF3A3A6;">interaction move forward</mark>**, not stall, offer options.


### Confirmation design
- <mark style="background: #FFF3A3A6;">Reflect</mark> <mark style="background: #FFF3A3A6;">what system</mark> <mark style="background: #FFF3A3A6;">thinks heard,</mark><mark style="background: #FFF3A3A6;"> proportional</mark>, <mark style="background: #FFF3A3A6;">depend on confidence</mark>, help catch misunderstandings early. 


## Responsible AI
- <mark style="background: #FFF3A3A6;">Approach</mark>, encompasses <mark style="background: #FFF3A3A6;">dev</mark>,<mark style="background: #FFF3A3A6;"> deployment</mark>, responsible use, AI systems, <mark style="background: #FFF3A3A6;">focused </mark>on <mark style="background: #FFF3A3A6;">stakeholders</mark>+<mark style="background: #FFF3A3A6;">system</mark>, considering legal+technical aspects, benefit stakeholders and environment. 

### RAI principle
- **<mark style="background: #FFF3A3A6;">Fairness</mark>**
- **<mark style="background: #FFF3A3A6;">Accountability</mark>**
- **<mark style="background: #FFF3A3A6;">Safety</mark>**
- **<mark style="background: #FFF3A3A6;">Transparency</mark>**
- **<mark style="background: #FFF3A3A6;">Privacy</mark>**
- **<mark style="background: #FFF3A3A6;">Sustainability</mark>**

### Why is RAI important
- LLM based CAI gained popularity, chai AI, airliner, cannot control response generated, can harm users, 
- Encourage HCD and responsibility. 


### RAI in CUI
- Privacy settings
- Transparency - give sources
- Fairness - chatbot in responses prevent amplification of biases when asked leading q's
- Safety - refuse help, unethical situation

### Designing persona of CAI system
- **<mark style="background: #FFF3A3A6;">Character traits</mark>**
- **<mark style="background: #FFF3A3A6;">Tone/voice</mark>**
- **<mark style="background: #FFF3A3A6;">Multimodal</mark>**
- **<mark style="background: #FFF3A3A6;">Multilingual support*</mark>*
- **<mark style="background: #FFF3A3A6;">Design of interface</mark>**

#### Persona design + responsible AI
| RAI principle  | How persona design affects it                                                                                      |
| -------------- | ------------------------------------------------------------------------------------------------------------------ |
| Fairness       | Persona can reinforce stereotypes (e.g., feminised voices for assistants)                                          |
| Transparency   | Persona can clarify or obscure AI limitations e.g., appearance of avatar or voice of conversational agent          |
| Safety         | A confident persona may mislead a user; a cautious persona can prevent harm                                        |
| Privacy        | Persona may influence how willing users are to disclose sensitive info.                                            |
| Accountability | Persona can signal boundaries                                                                                      |
| Sustainability | Professional persona may use fewer tokens for concise comms compared to friendly persona that relies on anecdotes. |



## Regulation dilemma
- **<mark style="background: #FFF3A3A6;">Too early</mark>**
	- Stifle innovation = 'regulation chill', limit scope of benefit
	- Legal frameworks becomes obsolete
- **<mark style="background: #FFF3A3A6;">Too late</mark>**
	- Delaying<mark style="background: #FFF3A3A6;"> allows harmful practices</mark> become<mark style="background: #FFF3A3A6;"> entrenched, </mark><mark style="background: #FFF3A3A6;">power dynamics</mark>, <mark style="background: #FFF3A3A6;">privacy violations</mark>; difficult to react to.

### Precautionary principle  vs Innovation Principle
- **<mark style="background: #FFF3A3A6;">Precationary principle</mark>**
	- Proactive regulation, prevent harm, safeguard public welfare
- **<mark style="background: #FFF3A3A6;">Innovation principle</mark>**
	- Inverse, focus advancement, regulate after

### UK approach, regulating AI
- **<mark style="background: #FFF3A3A6;">Sector-specific</mark>** strat, avoid overarching law
- Emphasise <mark style="background: #FFF3A3A6;">existing regulatory bodies</mark>, pro-innovation, while safeguarding
- **<mark style="background: #FFF3A3A6;">ICO</mark>** - data protection
- **<mark style="background: #FFF3A3A6;">Ofcom</mark>**
- **<mark style="background: #FFF3A3A6;">Financial conduct authority(FCA)</mark>**
- **<mark style="background: #FFF3A3A6;">Competition+markets authority(CMA)</mark>** - fair market
- **Medicines + healthcare products regulatory agency(MHRA)**

### Online safety Act
- <mark style="background: #FFF3A3A6;">Comprehensive measures</mark>, safeguard, particularly AI algorithmic systems
- Mandates <mark style="background: #FFF3A3A6;">safeguard</mark>s, <mark style="background: #FFF3A3A6;">risk assessments, </mark><mark style="background: #FFF3A3A6;">mitigate harms associated</mark> with **<mark style="background: #FFF3A3A6;">automatic content moderation</mark>** and recommendation.
- Covers <mark style="background: #FFF3A3A6;">user-generated content</mark>


### Data protection Act 2018 & UK GDPR
- <mark style="background: #FFF3A3A6;">DPA</mark>, cornerstone, regulated data processing
- Incorporate <mark style="background: #FFF3A3A6;">GDPR elements</mark>
- **<mark style="background: #FFF3A3A6;">Transparency in data processing</mark>**
- **<mark style="background: #FFF3A3A6;">Fairness</mark>(in AI systems)**
	- E.g., no discrim, hiring algos
- **<mark style="background: #FFF3A3A6;">Accountability</mark>**
	- Must comply
- **<mark style="background: #FFF3A3A6;">Data minimisation principle</mark>**
- **<mark style="background: #FFF3A3A6;">Rights of data subjects</mark>**

### Equality Act 2010 + Algorithmic bias
<mark style="background: #FFF3A3A6;">- Ai systems deployed, sensitive areas, adhere principles, non-discrimination, e.g., policing, housing, credit
- Algorithmic bias, arise, flawed training data/biased algorithms, risk perpetuating inequalities</mark>


### UK AI regulation
- Leverage existing regulatory bodies
- Just has a regulation framework
	- Grounded in **<mark style="background: #FFF3A3A6;">five foundational principles</mark>**
		1. **<mark style="background: #FFF3A3A6;">Safety</mark>**
		2. **<mark style="background: #FFF3A3A6;">Transparency</mark>**
		3. **<mark style="background: #FFF3A3A6;">Fairness</mark>**
		4. **<mark style="background: #FFF3A3A6;">Accountability</mark>**
		5. **Provide <mark style="background: #FFF3A3A6;">mechanism</mark> for <mark style="background: #FFF3A3A6;">contestability</mark>**

### EU AI act
- Structured regulatory framework, categorise AI system based on risk level.

#### Why EU regulated early
- **Desire for global leadership**
- **Public pressure over AI harms**
- **Fear of unregulated AI in safety-critical sectors**

#### Structure of EU AI act
- Categorises - 4 tiers, based on risk-level
	1. **Unacceptable risk**
		Banned: biometric identification, manipulative AI, social scoring by govs
	2. **High-risk** lots of regulations
		Medical diagnostic tools
		Recruitment tools - fairness
		Education - transparency + accuracy
		Essential public services - fairness
	3. **Limited risk** transparency obligations
		Adhere transparency rules e.g., chatbots, disclose AI nature, emo recog systems, notify that active
	4. **Minimal risk - unregulated**
		Game AI, spam filters, few to no new regulatory requirements. 

#### Enforcement of EU AI Act
- Establishes enforcement framework, regulate AI compliance
	1. **<mark style="background: #FFF3A3A6;">Oversight mechanism</mark>**
		National supervisory authorities, oversee compliance.
	2. **<mark style="background: #FFF3A3A6;">Financial penalties</mark>**
		Fines for violations, 7% of global turnover, or 35 million

### Implications; AI devs
<mark style="background: #FFF3A3A6;">- **Document sys design, training data**
- **Conduct bias + safety assessments**
- **Provide transparency to users</mark>**



## Ethics by design
- Integration, ethics, into design +dev of tech, mostly AI

### Core principles of EbD
- **Human centricity**
	- Emphasises needs, values, experience, userbase, welfare.
- **Transparency**
	- How functions
- **Accountability**
	- Identifying responsibility framework
- **Inclusivity**
	- Emphasise important, engaging, diverse range, stakeholders.

### Value sensitive design
- Methodological framework, systematically, accounts, human vals, in design of tech systems
	1. **Conceptual investigations** - define values
	2. **Empirical investigations** - data
	3. **Technical investigations** -solutions
	4. **Balancing values** - stakeholder engage
	5. **Iterative process** -refine
- **Strengths**
	- Promote ethical awareness in design
	- Encourage stakeholder engagement, more holistic
- **Weaknesses**
	- Operationalisation difficulties; theoretical vals-> design principles, difficult
	- Conflict values
- Parental controls use case

### Privacy by Design
- Proactive framework, privacy considerations, core principle.

##### Core principles
- **<mark style="background: #FFF3A3A6;">Proactive not reactive</mark>**
	- Anticipate privacy issues before occur. 
- **<mark style="background: #FFF3A3A6;">Privacy is default</mark>**
	- Design private from outset
- **<mark style="background: #FFF3A3A6;">User-centric</mark>**
	- Have control, over personal data, options for consent, access etc.

Strengths = reduce risk of data breaches, legal reqs
Weaknesses = complex, hard to implement in existing, trade-off with functionality

Case = google pay, apple pay, firefox


### Participatory design in AI development
- Emphasise,<mark style="background: #FFF3A3A6;"> active involvement, all stakeholders</mark>, align needs + vals

#### Core principles
- **Collaboration**
	- Collaborate designers + users
- **Empowerment**
	- Involves giving voice, marginalised group
- **Contextual understanding**
	- Crucial participatory design, deeply engage with envs + challenges faced by users.
Critiques = <mark style="background: #FFF3A3A6;">groupthink, marginalisation, resource-intensive commitment. </mark>

### Responsible Research and Innovation
- <mark style="background: #FFF3A3A6;">Framework, AREA</mark>, consider societal impacts, according structure, integrate ethical considerations into research process; innovation becomes result of ethical processing. 

#### Anticipation
- Proactive, approach, foresee risks with tech advancements.
#### Inclusion
- Involve stakeholders, communities, policy makers, in innovation process.
#### Reflexivity
- Req resaerchers, reflect, ethical implications of work, across social context; via **responsiveness,** adapt methods + approaches based on ethical consideration.'

Critiques include <mark style="background: #FFF3A3A6;">resitance from trad paradigms, inconsistency of implementation, conflicting stakeholder interests, short-term focuses overshadow ethics, resource constraints. 
</mark>
### Proactive ethics vs Reactive Ethics