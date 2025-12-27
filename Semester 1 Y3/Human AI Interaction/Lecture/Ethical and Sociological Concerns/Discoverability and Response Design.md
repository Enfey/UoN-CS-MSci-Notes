# Interacting with VUIs
- **Human Input** - poses 2 questions
	1. How do you know what the interface is capable of?
	2. How does one invoke these capabilities?
- **VUI Output** - poses 2 questions
	1. What if invocation doesn't meet human's intent?
	2. How can this be shaped to enable 'good' next turns?
- These are problems of **initiating interaction** and **response design**, respectively.

## Initiating Interaction
- **Assumption**: people can begin interaction with a VUI
- **Problem**: knowing how to discover the capabilities of the VUI
- **Design goals**
	- Intuitive 
	- Discoverable
- How does one achieve this?
### Discoverability
> Refers to the ease with which users can find and execute the system's available features and commands.

Particularly challenging in VUI context as often no ability to offer visual cues and menus, usually purely audio based making it difficult for users to know what they can or cannot say. 

An **affordance** refers to the action possibilities an environment or object offers to an individual. In design, affordance has a narrow meaning; it refers to possible actions that an actor can readily perceive. *Actions are easily discoverable only if they are perceivable.*

#### Discoverability: GUI vs VUI
Humans are generally better at **recognition** than **recall** according to cognitive psychology.
- **Recognition** - "I see the option and recognise it"
- **Recall** - "I have to remember what options exist without seeing them"
**Nielsen's usability heuristic** says:
> "Recognition rather than recall. Minimising the user's cognitive load by making elements, actions, and options visible. "

In voice interfaces, this principle is violated more often because nothing is visible. 

#### The discoverability challenge
From Kirschthaler et al (2020) (aimed to measure how different discoverability strategies affected UX and task performance via a Wizard-of-Oz study):
- Difficult for users to find and execute available functions in a user interface
- The invisibility and ephmerality of speech exacerbate this challenge with voice interfaces.
This makes discovering features difficult and can hurt:
- Task success
- Efficiency
- User satisfaction

**Study task:** Order two different dishes from a local takeaway restaraunt using a voice interface ("Feedme").
**Study conditions:** 
	1. Automatic help
	2. Requested help
	3. Baseline/no help
**Study flow:** 
	1.Choose cuisine
	2.Choose restaurant
	3.Choose dietary options
	4. Choose a dish
	5. Checkout

##### Discoverability support
In the study, there were three discoverability conditions:
1. Automatic help
	The system *proactively* tells users what they can say. 
	For example:
		**VUI:** “What would you like to eat? You can say Italian, American, or Japanese.”
2. Requested help
	The system only gives help when asked
	For example:
		**User:** “What can I say?”  
		**VUI:** “You can say Italian, American, or Japanese.”
	This relies more on **recall**, but help is still available.
3. Baseline/no help
	The system gives no guidance unless explicitly designed into the flow. Users must guess what commands work.

The measures used in the study include:
- **Performance metrics**
	- Time per task
	- Turns per task 
	- Completed stages
	- Errors per task
- **Perceived usability**
	- System usability scale
	- Post-task feedback discussion

##### Findings
- Automatic help performed the best overall, and was significantly better than the baseline in all measures
- Performance in the requested strategy varied, some users benefitted, while others didn't even ask for help when confused.
- Turns per task was the only metric for which a significant different between requested and automatic strategies was found.
##### Design takeaways
- Ideas of varying the discoverability strategy based on UX over time e.g., adopting automated prompts for new devices, new voices, or periodically to encourage experimentation, and then easing off to explicit requests after initial use.

## Response design
Martin Pocheron and colleagues conducted an **ethnographic, in-the-wild study:**
- Gave **Amazon Echo** devices to **5 households**
- Deployed for **one month**
- Used a **Conditional Voice Recorder**:
	- Records **1 min before** and **1 min after** the wake word.
	- Captures real, natural interaction (including frustration, confusion, laughter)
- This allowed the study of real conversational breakdowns. 
	![](Pasted%20image%2020251226202829.png)
### Findings
- Voice interaction is messy, people repeat themselves, change phrasing, talk over each other, laugh, get frustrated, and don't know how to discover/wield system capabilities
- The study found **30-50%** failure rates in everyday interactions
- Failures include:
	- Misrecognition
	- Misinterpretation
	- Wrong domain
	- Ambiguous intent (the above is intertwined with this)
- The problem is not just the failure, it is how the system recovers from failure via its subsequent responses. 
- Responses are not just outputs, they are resources users use to figure out what to do next. 
	"Sorry, I can't find the answer to the question I heard"
	Response seen above.
	- No clue what went wrong 
	- No hint what would word
	- No path forward
	- Displays confidently that the system had the incorrect intent
	A good response would explain the problem, show understanding, and offer next steps
### Recovery, Repair, and Progressivity
Core interaction design concepts:
- **Recovery**
	Helping the user recover from an error. (say what went wrong.)
	Instead of "I didn't understand" a prompt like "I couldn't find a quiz called 'family quiz'" would have supported recovery, without directly providing the tools to repair a request, simply aids.
- **Repair**
	Helping the user fix their request (say what the system doesn't understand)
	For example "You can say 'start a trivia game' or 'play Beat the Intro'" to teach the system's language.
- **Progressivity**
	Helping the interaction **move forward** and not stall to increase chance of task completion (offer options).
### Confirmation design
**Good confirmations**
- Reflect what the system *thinks* it heard
- Are proportional (don't over confirm everything)
- Depend on system confidence
	“I think you said ‘play Beat the Intro’. Is that right?”
- This design helps users catch misunderstandings early, and trust the system more. 