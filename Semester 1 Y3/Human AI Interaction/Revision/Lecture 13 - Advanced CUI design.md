## Branching + Disambiguation
### Approaches to branching
> Process, dialogue system chooses diff next actions based on user input.

Approaches to reduce error.
- **<mark style="background: #FFF3A3A6;">Constrained Responses</mark>**
- **<mark style="background: #FFF3A3A6;">Open speech</mark>**
- **<mark style="background: #FFF3A3A6;">Categorisation of input</mark>**
- **<mark style="background: #FFF3A3A6;">Wildcards + logical expressions</mark>**

#### Constrained responses
- Design prompt, constrain response, so response, likely contain, single keyword e.g., "tell me name of restaurant" vs "where do you want to eat"
- <mark style="background: #FFF3A3A6;">Good strategy,</mark> use <mark style="background: #FFF3A3A6;">N best-lists</mark> based on users, following constrained results. 
	- N best list, <mark style="background: #FFF3A3A6;">ordered list</mark> of <mark style="background: #FFF3A3A6;">top</mark> $N$ most likely<mark style="background: #FFF3A3A6;"> interpretations</mark>
	- <mark style="background: #FFF3A3A6;">Iterate over list</mark> <mark style="background: #FFF3A3A6;">until user confirms </mark>-<mark style="background: #FFF3A3A6;"> list comes from intent</mark>/<mark style="background: #FFF3A3A6;">slot parsing</mark>.
#### Open speech
- <mark style="background: #FFF3A3A6;">Allow for flow</mark>, <mark style="background: #FFF3A3A6;">when response</mark>, <mark style="background: #FFF3A3A6;">not need to be processed</mark>
	- Provide <mark style="background: #FFF3A3A6;">general reply</mark>, <mark style="background: #FFF3A3A6;">similar </mark>to <mark style="background: #FFF3A3A6;">generic confirmatiopn</mark>
- Use, obtain info, 'natural', 'conversational way'
- 'Normal way' - not restricted, use when no branching basically.
#### Categorisation of input
- <mark style="background: #FFF3A3A6;">Categorising user input</mark>
- E.g., emotions
- <mark style="background: #FFF3A3A6;">Works for cases</mark> where no <mark style="background: #FFF3A3A6;">need</mark> to confirm exactly what user said, just <mark style="background: #FFF3A3A6;">acknowledge category</mark> e.g., intent match.
#### Wildcards + logical expressions
- Wildcard = placeholder, can <mark style="background: #FFF3A3A6;">pattern match</mark> <mark style="background: #FFF3A3A6;">one word,</mark> many words, repeated words, <mark style="background: #FFF3A3A6;">need fewer rules</mark> in <mark style="background: #FFF3A3A6;">rule-based</mark> CUI.
- <mark style="background: #FFF3A3A6;">Logical expression</mark> = <mark style="background: #FFF3A3A6;">combine keywords</mark> + <mark style="background: #FFF3A3A6;">phrases </mark>into <mark style="background: #FFF3A3A6;">meaning based rules</mark>
- <mark style="background: #FFF3A3A6;">forgot OR lost OR don't remember</mark>
	Matches:
	"I forgot my password"
	"I lost my password"
	"I don’t remember my password"


### Disambiguation
<mark style="background: #FFF3A3A6;">Different situations</mark> require <mark style="background: #FFF3A3A6;">disambiguation</mark>; <mark style="background: #FFF3A3A6;">not enough info</mark>, <mark style="background: #FFF3A3A6;">too much info</mark> when only one piece expected. 

#### Disambiguation: not enough info
- When there is <mark style="background: #FFF3A3A6;">more than one kind</mark> e.g., whats the weather in springfield, one in every state.
- <mark style="background: #FFF3A3A6;">When information omitted</mark>
	- I'd like a large
- When <mark style="background: #FFF3A3A6;">intent is unclear</mark>
	- "Need help with internet"
- <mark style="background: #FFF3A3A6;">Reprompt</mark>
#### Disambiguation: more than one piece of information
- <mark style="background: #FFF3A3A6;">More than one piece of info</mark>
- **<mark style="background: #FFF3A3A6;">User response ambiguous</mark>**: strategy; ask user, <mark style="background: #FFF3A3A6;">pick more important one</mark>
- **<mark style="background: #FFF3A3A6;">User's request is ambiguous</mark>**: strategy, ask user, <mark style="background: #FFF3A3A6;">pick one of the options. </mark>
	- E.g., call sally -> sure, phone or work.

## Dialogue Management
- <mark style="background: #FFF3A3A6;">Make dialogue flow</mark>, <mark style="background: #FFF3A3A6;">flexible,</mark> effective, by <mark style="background: #FFF3A3A6;">managing conversational process</mark>, a lot of <mark style="background: #FFF3A3A6;">design principles</mark>
- Effective DM, adapt to user effectively. 
### Capturing Intent and objects
- Recognise both
	- Show me my calendar
- Intent and object, combined in concept of **utterances**
	- Utterances, how user says it, allow, map different ways, user may express themselves, to same intent
	- Objects can be captured as slots. 

### Handling negation
- Negative responses, important
- Express what don't want. 
- E.g., not very good -> matching on good
	- Not ideal
- Can be handled by LOGICAL EXPRESSIONS.


### Should VUI display what recognised
- May make transcription errors visible/audible
- Allow user recover from errors, not left guessing
- Good strategy, if utterance must be recognised to fulfill intent
- Can be distracting if not required though
- Strategy depends on where in process user is and context of app. 

### Sentiment analysis
- Process, computationally identifying + categorising opinions expressed in utterance
- NLP needed e.g., BOW from stemmed/lemmatising docs, compute rep for input, map, compute distance/similarity from inp to 'sentiment docs'
- In VUIs, capturing sentiment, useful, generate appropriate responses, on fly.

### TTS vs recorded human speech
- TTS much better now
	- Speech Synthesis Markup Language, can be used to tune **prosody** (pitch, rate(speed), volume) etc.
- Recording voice talent, can make VUI sound more natural, but costly, not flexible, cumbersome. 

### Speaker verification
- Biometric auth, voice ID

### Context
- Take advantage, basic context, make VUI seem smarter, track stages required, complete transaction
- Coreference resolution, facilitated too.

### Personalisation
- Use user's name and other info, personalise chatbot prompt
- Greeting user in time zone
- Referring last log on
- Use location
### Advanced multimodal
- Hybrid stats, voice + visual output
- Could also have hybrid strats of voice+visual input
- Or VUI embedded in website

### Bootstrapping datasets
- Build own lang models
	- Website data
	- Call center data
	- Data collection, whemn there isn't any available data, ask ppl questions that VUI would ask, transcribe, simulate VUI, also crowdscourcing
## Conclusions
 - VUI/CUI not solved problem, huge challenges around gap between how ppl talk, capabilities of ASR and NLU to recog intent, respond appropriately
 - A lot designers can do to make CUI/VUI effective
	 - Dialogue management, overarching concept
		 - Disambiguation, error recovery, negation, personalisation discoverability, multimodal, TTS, etc
		 - Overall, DM, help user, provide expected/well-formatted input via good prompt and response design, ensure task completion. 


