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
### <mark style="background: #FFF3A3A6;">Capturing Intent and objects</mark>
- <mark style="background: #FFF3A3A6;">Recognise both</mark>
	- Show me my calendar
- Intent and object, <mark style="background: #FFF3A3A6;">combined</mark> in concept of **<mark style="background: #FFF3A3A6;">utterances</mark>**
	- Utterances, how user says it, allow, <mark style="background: #FFF3A3A6;">map different ways</mark>, <mark style="background: #FFF3A3A6;">user </mark>may <mark style="background: #FFF3A3A6;">express themselves</mark>, to <mark style="background: #FFF3A3A6;">same intent</mark>
	- <mark style="background: #FFF3A3A6;">Objects</mark> can be <mark style="background: #FFF3A3A6;">captured</mark> as <mark style="background: #FFF3A3A6;">slots. </mark>

### <mark style="background: #FFF3A3A6;">Handling negation</mark>
- Negative responses, important
- <mark style="background: #FFF3A3A6;">Express</mark> <mark style="background: #FFF3A3A6;">what</mark> <mark style="background: #FFF3A3A6;">don't want</mark>. 
- E.g., not very good -> matching on good
	- Not ideal
- Can be handled by <mark style="background: #FFF3A3A6;">LOGICAL EXPRESSIONS.</mark>


### <mark style="background: #FFF3A3A6;">Should VUI display what recognised</mark>
- May <mark style="background: #FFF3A3A6;">make transcription errors visible</mark>/audible
- Allow user recover from errors, <mark style="background: #FFF3A3A6;">not left guessing</mark>
- <mark style="background: #FFF3A3A6;">Good strategy</mark>, <mark style="background: #FFF3A3A6;">if</mark> utterance <mark style="background: #FFF3A3A6;">must</mark> be<mark style="background: #FFF3A3A6;"> recognised</mark> to<mark style="background: #FFF3A3A6;"> fulfill</mark> <mark style="background: #FFF3A3A6;">intent</mark>
- Can be <mark style="background: #FFF3A3A6;">distracting</mark> if not required though
- <mark style="background: #FFF3A3A6;">Strategy depends</mark> on where in process user is and context of app. 

### <mark style="background: #FFF3A3A6;">Sentiment analysis</mark>
- Process, computationally identifying + categorising opinions expressed in utterance
- NLP needed e.g., BOW from stemmed/lemmatising docs, compute rep for input, map, compute distance/similarity from inp to 'sentiment docs'
- In VUIs, capturing sentiment, useful, <mark style="background: #FFF3A3A6;">generate appropriate responses</mark>, on fly.

### <mark style="background: #FFF3A3A6;">TTS vs recorded human speech</mark>
- TTS much better now
	- Speech Synthesis Markup Language, can be used to tune **prosody** (pitch, rate(speed), volume) etc.
- Recording voice talent, can make VUI sound more natural, but costly, not flexible, cumbersome. 

### <mark style="background: #FFF3A3A6;">Speaker verification</mark>
- Biometric auth, voice ID

### <mark style="background: #FFF3A3A6;">Context</mark>
- Take advantage, basic context, make VUI seem smarter, track stages required, complete transaction
- <mark style="background: #FFF3A3A6;">Coreference resolution,</mark> facilitated too.

###<mark style="background: #FFF3A3A6;"> Personalisation</mark>
- Use user's name and other info, personalise chatbot prompt
- Greeting user in time zone
- Referring last log on
- Use location
### <mark style="background: #FFF3A3A6;">Advanced multimodal</mark>
- Hybrid stats, voice + visual output
- Could also have hybrid strats of voice+visual input
- Or VUI embedded in website

### <mark style="background: #FFF3A3A6;">Bootstrapping datasets</mark>
-<mark style="background: #FFF3A3A6;"> Build own lang models</mark>
	- Website data
	- Call center data
	- Data collection, whemn there isn't any available data, ask ppl questions that VUI would ask, transcribe, simulate VUI, also crowdscourcing
## Conclusions
 - VUI/CUI <mark style="background: #FFF3A3A6;">not solved problem</mark>, huge challenges around <mark style="background: #FFF3A3A6;">gap between how ppl talk</mark>, capabilities of <mark style="background: #FFF3A3A6;">ASR</mark> and <mark style="background: #FFF3A3A6;">NLU</mark> to <mark style="background: #FFF3A3A6;">recog intent</mark>, respond appropriately
 - A lot designers can do to make CUI/VUI effective
	 - <mark style="background: #FFF3A3A6;">Dialogue management</mark>, overarching concept
		 - <mark style="background: #FFF3A3A6;">Disambiguation</mark>, error recovery, negation, personalisation discoverability, multimodal, TTS, etc
		 - Overall, DM, help user, provide expected/well-formatted input via good prompt and response design, ensure task completion. 


