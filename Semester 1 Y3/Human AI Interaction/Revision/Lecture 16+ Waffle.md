## Discoverability
> ease, which users, find, execute, system commands/features

Challenging, VUI context, no visual cues

An **affordance** = action possibilities environment offers. Affordance has narrow meaning; **possible actions actor can perceive**.


### Discoverability GUI vs VUI
- **Recognition**
- **Recall**
- Recognition better, minimise cognitive l,oad
- Voice interface, principle violated

### Discoverability challenge
- Invisibility+ephmerality of speech, exacerbate issues, difficult users find + exec available functions
- Can hurt:
	- Task success
	- Efficiency
	- User satisfation
- **Study task** - order 2 diff dishes from local takeaway 
	- **Study conditions**
		- Automatic help
		- Requested help
		- No help(baseline)
	- Measured
		- Perf metrics
			- Time per task
			- Turns per task
			- Errors per task
			- Completed stages
		- Perceived usability
			- SUS
			- Post-task discussion feedback
	- Automatic help = best, performance in requested varied, baseline worst, turns per task only metric for which sig diff found between requested and automatic.
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
- Voice interaction = messy, people repeat selves, change phrasing, talk over each other
- 30-50% failure rates in everyday interactions
	- Misrecognition
	- Misinterpretation
	- Wrong domain
	- Ambiguous intent
- Need good error recovery, explain problem, offer steps.

### Recovery, repair, progressivity
- **Recovery**
	- Recover from error, say what went wrong e.g., instead of "I didn't understand" a prompt like "I couldn't find a quiz called 'family quiz'"
- **Repair**
	- Help user fix their request, give resolution "You can say start a quiz", teach the system's language
- **Progressivity**
	- Help **interaction move forward**, not stall, offer options.


### Confirmation design
- Reflect what system thinks heard, proportional, depend on confidence, help catch misunderstandings early. 


## Responsible AI
- Approach, encompasses dev, deployment, responsible use, AI systems, focused on stakeholders+system, considering legal+technical aspects, benefit stakeholders and environment. 

### RAI principle
- **Fairness**
- **Accountability**
- **Safety**
- **Transparency**
- **Privacy**
- **Sustainability**

### Why is RAI important
- LLM based CAI gained popularity, chai AI, airliner, cannot control response generated, can harm users, 
- Encourage HCD and responsibility. 


### RAI in CUI
- Privacy settings
- Transparency - give sources
- Fairness - chatbot in responses prevent amplification of biases when asked leading q's
- Safety - refuse help, unethical situation

### Designing persona of CAI system
- **Character traits**
- **Tone/voice**
- **Multimodal**
- **Multilingual support**
- **Design of interface**

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
- **Too early**
	- Stifle innovation = 'regulation chill', limit scope of benefit
	- Legal frameworks becomes obsolete
- **Too late**
	- Delaying allows harmful practices become entrenched, power dynamics, privacy violations; difficult to react to.

### Precautionary principle  vs Innovation Principle
- **Precationary principle**
	- Proactive regulation, prevent harm, safeguard public welfare
- **Innovation principle**
	- Inverse, focus advancement, regulate after

### UK approach, regulating AI
- **Sector-specific** strat, avoid overarching law
- Emphasise existing regulatory bodies, pro-innovation, while safeguarding
- **ICO** - data protection
- **Ofcom**
- **Financial conduct authority(FCA)**
- **Competition+markets authority(CMA)** - fair market
- **Medicines + healthcare products regulatory agency(MHRA)**

### Online safety Act
- Comprehensive measures, safeguard, particularly AI algorithmic systems
- Mandates safeguards, risk assessments, mitigate harms associated with **automatic content moderation** and recommendation.
- Covers user-generated content


### Data protection Act 2018 & UK GDPR
- DPA, cornerstone, regulated data processing
- Incorporate GDPR elements
- **Transparency in data processing**
- **Fairness(in AI systems)**
	- E.g., no discrim, hiring algos
- **Accountability**
	- Must comply
- **Data minimisation principle**
- **Rights of data subjects**

### Equality Act 2010 + Algorithmic bias
- Ai systems deployed, sensitive areas, adhere principles, non-discrimination, e.g., policing, housing, credit
- Algorithmic bias, arise, flawed training data/biased algorithms, risk perpetuating inequalities


### UK AI regulation
- Leverage existing regulatory bodies
- Just has a regulation framework
	- Grounded in **five foundational principles**
		1. **Safety**
		2. **Transparency**
		3. **Fairness**
		4. **Accountability**
		5. **Provide mechanism for contestability**

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
	1. **Oversight mechanism**
		National supervisory authorities, oversee compliance.
	2. **Financial penalties**
		Fines for violations, 7% of global turnover, or 35 million

### Implications; AI devs
- **Document sys design, training data**
- **Conduct bias + safety assessments**
- **Provide transparency to users**



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
- **Proactive not reactive**
	- Anticipate privacy issues before occur. 
- **Privacy is default**
	- Design private from outset
- **User-centric**
	- Have control, over personal data, options for consent, access etc.

Strengths = reduce risk of data breaches, legal reqs
Weaknesses = complex, hard to implement in existing, trade-off with functionality

Case = google pay, apple pay, firefox


### Participatory design in AI development
- Emphasise, active involvement, all stakeholders, align needs + vals

#### Core principles
- **Collaboration**
	- Collaborate designers + users
- **Empowerment**
	- Involves giving voice, marginalised group
- **Contextual understanding**
	- Crucial participatory design, deeply engage with envs + challenges faced by users.
Critiques = groupthink, marginalisation, resource-intensive commitment. 

### Responsible Research and Innovation
- Framework, AREA, consider societal impacts, according structure, integrate ethical considerations into research process; innovation becomes result of ethical processing. 

#### Anticipation
- Proactive, approach, foresee risks with tech advancements.
#### Inclusion
- Involve stakeholders, communities, policy makers, in innovation process.
#### Reflexivity
- Req resaerchers, reflect, ethical implications of work, across social context; via **responsiveness,** adapt methods + approaches based on ethical consideration.'

Critiques include resitance from trad paradigms, inconsistency of implementation, conflicting stakeholder interests, short-term focuses overshadow ethics, resource constraints. 

### Proactive ethics vs Reactive Ethics