## User Testing for VUIs
Special considerations must be made.
- Do users **understand they can talk to the system**? DO they know how?
	- Discoverability problem; prompt design.
- Does the VUI **understand the way people actually talk to it**?
	- What are the kinds of things people say, and words used, VUI needs to recognise them and recognise intent
- Is the dialogue management effective
	- Do people get things done with it they're supposed to? How well do implemented strategies e.g., error recovery, confirmation, disambiguation work. 

## User testing
### Why testing with real users?
User centred design
- Find problems early on in process, fix them. 
- Most technology is designed to be **used by and useful** for a particular audience, so testing should be done with them too for effective delivery
- People draw on **prior experience** when interacting with technology; can help or hinder, only find out by interacting
- Testing helps improve product, therefore make more money etc
- People can have **exper/local knowledge** that helps design better VUI, especially if target audience. 

### Testing for what purpose?
What is user testing for?
- To measure **UX** or **task performance**(response times, accuracy)
	- Subjective e.g., ratings /objective measurement (elapsed time)
	- Self-reported vs behavioural/observational data e.g., number of tasks
- **Informs design decisions** (formative evaluation)
	- Early on, or mid stage
- **Evaluate final prototype** (summative evaluation)
- Report **success measures** to clients, write a paper, have other extrinsic reasons. 

### User Study
- **Goal**
	- What is goal/purpose for study
	- Can be driven by research questions, hypotheses, exploratory
	- Study UX, performance etc.
- **Tasks**
	- What are you asking the user to do?
- **Participants**
	- Ethics, recruitment, instructions, reimbursement, sampling etc
- **Data collection and analysis**
	- How and what data are collecting (interviews, surveys, measurement)
	- Does it measure what want to measure(validity)
	- How is the data being analysed (quant vs qual)

### Task Definition
- Designed to exercise **parts of the system** wish to test
- Focus on **primary dialogue paths**
	- Features that are likely to be used **frequently**
	- Tasks in areas of **high risks**
	- Tasks that address the major goals and design criteria identified during requirements definition.
- Write the **task definitions** carefully to avoid biasing the participant
- **Describe the goal** of the task without mentioning command words or strategies for completing said task to elicit realistic interaction.

### Task Order
- To avoid order effects (e.g., primacy and recency effects)
- Randomise tasks if possible, e.g., use latin square design, each task in every position. 
	![](Pasted%20image%2020251225014640.png)
- **Counterbalance** order if using conditions (different versions of a task for example) to prevent a condition's results being confounded by its position. 

### Participants
- **Characteristics**
	- Demographics, experience of use, practicality etc.
- **Sample and population**
	- Stratified sample
		- Sample of certain chracterictics
	- Want to ensure sample is representative, need stratified, and need large N to allow generalisable results.
- **How many is enough?**
	- Jakob Nielsen's advice: test with 5 users or less (2 users for low-fidelity prototypes) and do many iterations of testing (at least 3 rounds of testing).
	- For fewer iterations test with 8-10 users for prototypes, and 15-20 users for finished products. If you can, or need to iterate then a second round of testing should suffice.

### Data collection
- Questionnaires/surverys/interviews to gather self-report data
	- Lots of techniques, standardised instruments exist e.g., SUS survey used in UX research
	- Tools to make own, open/closed ended Q, liker scale, rankings, scores etc
-  Measurements, video/audio to gather observational data:
	- Quantitative: Errors, time, completions, number of words etc. 
	- Qualitative: Audio recordings of dialogs / interactions with VUIs

### Data Analysis
- **Quantitative**
	- Hypothesis testing
	- Descriptive and inferential statistics
		- Depend on type of data and its distribution
	- Algorithmic/mathematical e.g., precision and recall
- **Qualitative**
	- Thematic analysis (finding the 'themes' in data)
	- Conversation analysis(focused on talk-in-interaction)
	- Interaction analysis (multimodal)

## Early Stage and Usability Testing
### Early-Stage Testing
- Test concepts and dialogue flow early on in design process
- Table reads with **sample dialogues**
	- How does it sound? Repetitive? Or concise and natural?
	- Good for spotting too many transitions with the same conversational markers
- Initial reactions to **mock-ups**
- **Wizard of Oz** testing
	- Human behind the curtain simulates fully working system
	- Realistic but much cheaper/quicker to implement.
	- **Elicitation study** to learn what language/terms people use.
	- How does one create a **realistic simulation** of features of an NLU system like recognition accuracy?
		Human wizard's understanding makes simulation of NLU unrealistic, need **strict protocol** and the wizard's **rules** e.g.,
		- **If** the user says the right 'keywords' **AND** the request is complete, produce success response **ELSE** p;roduce error response
### Usability testing
- When you have **working app**
	Should have features you want to test in a fully working state
	May need to create a fake user account for this
	Parts may still be hardcoded 
- Can test the **dialogue flow** and **ease of use**
	- Not generally aimed at testing **recognition accuracy**
	- Usually in a lab setting ('controlled')
	- But can be done **remotely**
		- Many more potential users can take part online
- **Measure** both **performance** and **experience**
	- Likes and dislikes, but also task completion, errors, recovery etc.
	- Key measurements for testing VUI systems:
		- Accuracy and speed, cognitive effort, transparency/confusion, friendliness, and voice.
	- **Analysis**
		- **Identify pain points**: Where did users struggle? Dit they know when it was OK to speak? When things did go wrong, were they able to recover successfully?
		- **Recommendations**: rank issues by severity, create plan on when and how going to be able to fix them.
#### The Chatbot Usability Questionnaire
- Questionnaire specifically designed for measuring the usability of chatbots developed by researchers at Ulster University.
- CUQ designed to be comparable to SUS but is bespoke for chatbots and includes 16 items
- Measures: realism, engagement, friendliness, purpose, ease of use, understanding, relevance of responses, error handling etc. 
- Gives score out of 100 just like SUS. 

#### Settings
- Lab vs Real World
	- In-car testing - Simulators or the real thing
	- In the home. Need to balance potential intrusions with the benefits.
- Generally, field trial is the setting that this technology goes into, shows the messiness of technology in context, but ecologicaly validity is a key quality of research studies. 

## Prerelease and Pilot Testing

### Prerelease Testing
- **Dialogue Traversal Testing**
	- The purpose is to make sure that the system **accurately implements** the dialogue specification in complete detail.
	- Tests **all** transitions, error prompts, help prompts, or anything else that could happen **at any given state** in dialogue
	- Every universal and every error condition must be tested
		- E.g., try an out-of-grammar utterance to test the behaviour in response to a recognition reject, try silence to test no-speech timeouts, multiple successive errors within dialogue states etc
- **Load Testing**
	- Stress test of concurrent user sessions, determine outcome, will crash, slow to crawl etc. 

### Pilot Testing
- Define **success criteria**
	- Not just recognition accuracy
	- Talk to marketing, sales, support, other stakeholders e.g., 
		- 60% of users who start to make a hotel reservation complete it
		- Error rate for playing songs is less than 15%
		- Five hundred users download app in first month
- Other key **measures of success**:
	- **Task completion rates** - % of tasks completed
	- **Dropout rates** - inverse of above
	- **Time spent** - avg time per task, or overall in app
	- **Barge-in** - position and frequencies
	- **Speech vs GUI** - rate and positions
	- **No-speech timeouts** 
	- **No intent matches** - correct vs false rejects
	- **Errors** - types of errors, position, recovery
	- **Navigation** - back button use, menu items etc.
	- **Latency** - end of speech timeout, return of result

#### Logging
- Log measures of success e.g., 
	- Recognition result (including conf scores)
	- N-best list (if available (list of possible hypotheses))
	- Audio of user's utterance - for each state, including pre- and post-endpointed utterances
	- Errors 
	- State names
	- Latency
	- Barge-in info