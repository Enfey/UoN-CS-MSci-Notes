## User testing for VUIs
Special considerations made:
- Do users, understand, can talk to system? Do they know how
	- Discoverability problem
- Does VUI understand, way ppl talk to it?
	- What ppl say, words used, need to recognise them and intent
- Is DM effective?
	- Task completion, error recovery, disambiguation, branching, confirmation etc. 

## User testing
### Why testing w/ real users
UCD
- Find problems, early on in process, fix them
- Most tech, designed, used by, useful for, specific audience, test with them, for effective delivery
- Ppl, draw, on prior experience, when interacting, tech, can help/hinder; only find out via interacting.
- Testing, improve product, more money
- People, can have expert/local knowledge, design better VUI

### Testing: for what purpose?
- Measure UX/task performance
	- Subjective e.g., ratings vs Objective
	- Self-reported vs behavioural/observational
- Inform design decisions (**formulative evaluation**)
	- Early on, mid-stage
- Evaluate final prototype (**summative evaluation**)
- Report success measures, to clients, write a paper, etc.

### User study
- **Goal**
	- What is purpose/goal
	- Driven by research questions, hypotheses
	- Study UX, perf
- **Tasks**
	- What asking user to do
- **Ppts**
	- Ethics, recruitment, instruction, reimbursement, sampling
- **Data collection/analysis**
	- How/what data collecting (interviews, surveys)
	- Is it valid
	- How being analysed(quant vs qual)

#### Task definition
- Designed, exercise, parts of system, wish to test
- Focus on primary dialogue paths
	- Features, used freq
	- Tasks, areas, high risks
	- Tasks, address major goals/design criteria, ident in req def
- Write task definitions, carefully, avoid bias
- Describe goal of task, no mention, command words, elicit, real interaction

#### Task Order
- Avoid order effects (primacy, recency effects)
- Randomise tasks, **latin square design**, each task, in every pos.
	![](Pasted%20image%2020251225014640.png)
- **Counterbalance** order if using conditions (different versions of a task), prevent a condition's results being confounded by its position. 
	- Half do condition A, then B
	- Half do condition B, then A
	- Only use when conditions

#### Participants
- **Characteristics**
	- Demographics, experience of use, practicality
- **Sample+pop**
	- Stratified, accoring to certain characteristics
	- Need large enough N, generalise results
- **How many is enough**
	- Nielsen: test with 5 users or less, 2 for low-fidel prototypes, many iterations of testing e.g., 3
	- For fewer iterations, test w/ more users, up to 20 for finished products. 

#### Data collection
- Questionnaires/surveys/structured interviews, gather self-report data
	- Lots of techniques, e.g., SUS
	- Tools make own, open/closed ended, Q, liker scale, rankings, scores etc.
- Measurement, video/audio gather observational data
	- Quant: errors, time, completions, num words
	- Qual: audio recordings/interactions with VUIs

#### Data analysis
- Quant
	- Hypothesis testing
	- Descriptive+inferential statistics
		- Depend on type of data + distribution
	- Algorithmic/mathematical e.g., precision and recall
- Qual
	- Thematic analysis
	- Conversation analysis
	- Interaction analysis(multimodal)

### Early stage and Usability testing

#### Early stage testing
- Test concepts, dialogue flows, early on, design process
- **Table reads** w/sample with **sample dialogues**
	- How sound? Reptitive? Concise? Natural?
	- Good for spotting too many transitions
- Initial reactions to **mock-ups**
- Wizard of Oz
	- Human simulate fully working system, cheap, realistic
	- Elicitation study, learn what lang people use
	- How does one, realistically simulate features e.g., recog accuracy
		- Human wizard understanding, makes simulation, unrealistic, need strict protocol, wizard's rules.

#### Usability testing
- When have **working app**
	- Should have features, want to test, in fully working state, parts may still be hardcoded
- Can test dialogue flow, ease of use
	- Not aimed, testing perf
	- Done in lab setting
	- Can be done remotely?
- Measure both perf, and experience.
	- Cognitive effort, accuracy, speed, transparency, friendliness (VUI)
- Analysis
	- Identify pain points
	- Recommendations: rank issues by severity; plan how to fix and when

#### Chatbot usability questionnaire
- Questionnaire, designed, measure, **usability**
- CUQ, designed, comparable, SUS, 16 items
- Measures: **realism, engagement, friendliness, purpose, ease of use, understanding, relevance of responses, error handling**

#### Settings 
- Lab vs Real-World
- Generally, field trial, shows messiness, but ecological validity.

### Prerelease + Pilot testing
#### Prerelease tesing
- **Dialogue traversal testing**
	- Purpose: system accurately implement dialogue spec
	- Test all transitions, error prompts, help prompts
	- Try out of grammar utterances, try silence to test no-speech time outs.
- **Load testing**
	- Stress test, concurrent user sessions, determine outcome. 

#### Pilot testing
- Define **success criteria**
	- Not just recog accuracy
	- Talk martketers, sales, support, shareholders
		- 500 users first month, 60% users make hotel reservation after starting it
- Key measures of success:
	- **Task completion rate**
	- **Dropout rates** - inverse of above
	- **Time spent**
	- **Barge-in**
	- **Speech vs GUI**
	- **No-speech timeouts**
	- **No-intent matches**
	- **Errors**
	- **Navigational info**
	- **Latency**

#### Logging
- Log measures of success e.g., 
	- Recognition result (including conf scores)
	- N-best list (if available (list of possible hypotheses))
	- Audio of user's utterance - for each state, including pre- and post-endpointed utterances
	- Errors 
	- State names
	- Latency
	- Barge-in info
