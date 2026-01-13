# Conversational design
## Command + Control vs Conversation
Smartspeakers, operate, '<mark style="background: #FFF3A3A6;">one-shot interactions</mark>', rather than multi-turn convos. <mark style="background: #FFF3A3A6;">Wake word</mark> needed, explicitly <mark style="background: #FFF3A3A6;">invoke interaction</mark>, then listen, predetermined period

What does conversational mean?
	For CUI, <mark style="background: #FFF3A3A6;">conversational</mark>, involves, **<mark style="background: #FFF3A3A6;">prompt design</mark>**
	Design of system response, 'prompt for user'
	Involves,<mark style="background: #FFF3A3A6;">**conversational markers**</mark>, make <mark style="background: #FFF3A3A6;">interactions</mark>, more <mark style="background: #FFF3A3A6;">human-like</mark>

## Conversational markers
<mark style="background: #FFF3A3A6;"> Linguistic</mark>, elements, <mark style="background: #FFF3A3A6;">make interactions</mark>, <mark style="background: #FFF3A3A6;">natural.</mark>

**<mark style="background: #FFF3A3A6;">Timelines</mark>**: "First", "Halfway", "Finally"
**<mark style="background: #FFF3A3A6;">Acknowledgement</mark>**: "Got it", "Thanks"
**<mark style="background: #FFF3A3A6;">Positive Feedback</mark>**: "Good job", "Nice"
**<mark style="background: #FFF3A3A6;">Greetings</mark>**

## Conversational design examples
![[Pasted image 20251111193324.png]]


## Context Tracking CUIs
Maintain, <mark style="background: #FFF3A3A6;">conversational context</mark>,<mark style="background: #FFF3A3A6;"> across multi-turn</mark>, important, context = <mark style="background: #FFF3A3A6;">background info</mark>, <mark style="background: #FFF3A3A6;">prev interactions</mark>, situational awareness. <mark style="background: #FFF3A3A6;">No concrete def</mark>, can have many elems. 

![[Pasted image 20251111193955.png]]
	Coref resolution = identify expressions in text, referring to same entity, achievable through context or ML.

### Types of context

#### <mark style="background: #FFF3A3A6;">Conversational context</mark>
- What said earlier
- Topic discussed
#### <mark style="background: #FFF3A3A6;">Entity context</mark>
- Who/what being discussed
- Attributes, entities mentioned
- Relationships between entities
#### <mark style="background: #FFF3A3A6;">Task Context</mark>
- What user, trying, accomplish
- Where they are, multi-step process
- Completed vs pending actions
#### <mark style="background: #FFF3A3A6;">User context</mark>
- User prefs, history, location, timezone, name etc.
#### <mark style="background: #FFF3A3A6;">Environmental Context</mark>
- Time of day, location, device being used.


### Coreference resolution
Task: <mark style="background: #FFF3A3A6;">identifying</mark> when <mark style="background: #FFF3A3A6;">expressions</mark> in text/conversation <mark style="background: #FFF3A3A6;">refer </mark>to <mark style="background: #FFF3A3A6;">same entity</mark>; aspect of context tracking, focused, <mark style="background: #FFF3A3A6;">linking refs.</mark>

## Conversational design recommends
- <mark style="background: #FFF3A3A6;">Context</mark>
- <mark style="background: #FFF3A3A6;">Personalise responses</mark>
- <mark style="background: #FFF3A3A6;">Discoverability</mark>
- Be <mark style="background: #FFF3A3A6;">concise;</mark> provide <mark style="background: #FFF3A3A6;">clear,</mark> <mark style="background: #FFF3A3A6;">actionable</mark> <mark style="background: #FFF3A3A6;">prompts.</mark>
- <mark style="background: #FFF3A3A6;">Error recovery</mark>
- <mark style="background: #FFF3A3A6;">Set user expectations</mark>; transparent about capabilities
- Let use decide, how long conversation is. 


# CUI design process and tools
## Design Process 
- How <mark style="background: #FFF3A3A6;">ensure</mark> <mark style="background: #FFF3A3A6;">effective</mark> <mark style="background: #FFF3A3A6;">CUI design</mark>?
- Follow UX process
	- **<mark style="background: #FFF3A3A6;">User-centered</mark>(UCD)** or <mark style="background: #FFF3A3A6;">Human Centered Design</mark> (HCD).
		- <mark style="background: #FFF3A3A6;">HCD</mark> takes <mark style="background: #FFF3A3A6;">broad</mark>, <mark style="background: #FFF3A3A6;">empathetic approach</mark>, focuses on complex system problems, apply<mark style="background: #FFF3A3A6;"> human factor</mark>s + <mark style="background: #FFF3A3A6;">usability knowledge</mark>, enhance effectiveness. 
		- <mark style="background: #FFF3A3A6;">UCD </mark>focus on <mark style="background: #FFF3A3A6;">specific subset,</mark> incorporate elements of <mark style="background: #FFF3A3A6;">HCD,</mark> concentrate<mark style="background: #FFF3A3A6;"> specific tasks</mark> etc, more focused.
## Design tools
- Support, <mark style="background: #FFF3A3A6;">taking concepts</mark> -> <mark style="background: #FFF3A3A6;">prototypes</mark>
- Include:
	- <mark style="background: #FFF3A3A6;">Sample dialogues</mark>
	- <mark style="background: #FFF3A3A6;">Visual mockups</mark>
	- <mark style="background: #FFF3A3A6;">Flow</mark>
	- <mark style="background: #FFF3A3A6;">Prototyping tools</mark>
### Sample dialogues
- Write, <mark style="background: #FFF3A3A6;">sample convos</mark>, between CUI and users; prevents writing prompts in isolation, can lead to stilted experiences. Focus, <mark style="background: #FFF3A3A6;">5 most common CUI use cases</mark>,<mark style="background: #FFF3A3A6;"> start</mark> <mark style="background: #FFF3A3A6;">best-path scenarios</mark>, but do error ones too.
### Mockups
- <mark style="background: #FFF3A3A6;">Similar</mark> to <mark style="background: #FFF3A3A6;">wireframes</mark>, useful, <mark style="background: #FFF3A3A6;">hybrid CUIs</mark>, <mark style="background: #FFF3A3A6;">graphical components</mark>, combine with sample dialogue, create <mark style="background: #FFF3A3A6;">visual storyboard</mark>, excellent to <mark style="background: #FFF3A3A6;">present early concepts.</mark>
### Flow diagrams
- <mark style="background: #FFF3A3A6;">Decision tree style</mark> <mark style="background: #FFF3A3A6;">diagrams</mark>, show<mark style="background: #FFF3A3A6;"> paths, through CUI</mark>, limitation; <mark style="background: #FFF3A3A6;">lang interaction</mark> <mark style="background: #FFF3A3A6;">not hierarchical.</mark> 
# Confirmations
- <mark style="background: #FFF3A3A6;">Critical</mark>, CUI design, <mark style="background: #FFF3A3A6;">verifying user actions</mark>, before execution. Over-confirmation = problem 
## Considerations for confirmation strategy
1. **<mark style="background: #FFF3A3A6;">Consequences of errors</mark>**
	- What happen, if system, gets wrong, severity.
2. What **<mark style="background: #FFF3A3A6;">modalities</mark>** will system have to provide feedback
	Audio, text, visual, multiple etc.
3. **<mark style="background: #FFF3A3A6;">Screen size</mark>**
4. **<mark style="background: #FFF3A3A6;">What type of confirmation appropriate</mark>**
	Explicit, implicit, hybrid?
## Types of Confirmations
### <mark style="background: #FFF3A3A6;">Explicit</mark>
- Require user, t<mark style="background: #FFF3A3A6;">ake extra conversational turn</mark>, to confirm
	- "I think you want reminder to 'buy insurance before going skydiving next week.' is that correct?
### <mark style="background: #FFF3A3A6;">Implicit</mark>
- No additional turn needed to confirm action to be taken

### Confirmation methods
1. **<mark style="background: #FFF3A3A6;">Three-tiered confidence strategy</mark>**
	Adjust confirmation type, based on confidence level. 
	If >80%, implicit
	If <80% && >=45%, explicit
	If <45%, clarify
2. **<mark style="background: #FFF3A3A6;">Implicit confirmation</mark>**
	Build part of request into response, so user confident in answer.
3. **<mark style="background: #FFF3A3A6;">Generic confirmation</mark>**
	Universal responses, work, many contexts.
4. **<mark style="background: #FFF3A3A6;">Visual confirmation</mark>**
	Good, hybrid experiences, display calendar entry, on screen. 
5. **<mark style="background: #FFF3A3A6;">Nonspeech confirmation</mark>**
	Physical action = confirmation e.g., turn on lights, response = turning them on. 

# Error handling
- **"When talk to human, never unrecoverable error state" - Abi Jones**
- Principle, guide CUI error handling

## Traditional Error Handling
- **<mark style="background: #FFF3A3A6;">Reprompt</mark>**
	- Trad response, returned, if user not heard/understood.
- **<mark style="background: #FFF3A3A6;">Silence</mark>**
	- Work in much same way as reprompt, indicate issues
	- People respond to silent VUIs as indicating problems
	- <mark style="background: #FFF3A3A6;">Not explain error</mark>.

### Types of VUI errors
- <mark style="background: #FFF3A3A6;">No speech detected</mark>
	- Pre ASR error
	- No response returned
	- <mark style="background: #FFF3A3A6;">Provide vis indic</mark>, system listening, absence of indic, shows not listening, broken
- <mark style="background: #FFF3A3A6;">Speech detected</mark>,<mark style="background: #FFF3A3A6;"> nothing recognised</mark>
	- <mark style="background: #FFF3A3A6;">Transcription failure,</mark> didn't understand response
	- Handling strats:
		- Do nothing
		- <mark style="background: #FFF3A3A6;">Indicate system processed query</mark>, <mark style="background: #FFF3A3A6;">no response coming</mark>
		- Call out explicitly/reprompt
- <mark style="background: #FFF3A3A6;">Something recognised incorrectly</mark>
	- <mark style="background: #FFF3A3A6;">Transcription error</mark>
	- <mark style="background: #FFF3A3A6;">Unexpected/unintended response</mark>
	- <mark style="background: #FFF3A3A6;">Avoidable via confirmation</mark>
	- Partially incorrect transcriptions very frequent
		- <mark style="background: #FFF3A3A6;">Present partal response</mark>, <mark style="background: #FFF3A3A6;">allow user correct</mark>
		- Build error into response.
- <mark style="background: #FFF3A3A6;">Something recognised correctly</mark>, system <mark style="background: #FFF3A3A6;">does wrong thing </mark>with it
	- <mark style="background: #FFF3A3A6;">Semantic error</mark>
	- Unexpected/unintended response
	- Common, text-based CUIs too
	- <mark style="background: #FFF3A3A6;">Avoidable</mark> via <mark style="background: #FFF3A3A6;">disambiguation</mark>

### Reprompt strategies for error handling
- Instead of generic error messages, <mark style="background: #FFF3A3A6;">progressively provide more specific guidance</mark>
- General principles: <mark style="background: #FFF3A3A6;">don't blame user,</mark> design for diff user types

## Universal commands  and help and extras
Help has significance in VUIs, due to <mark style="background: #FFF3A3A6;">invisibility of affordances</mark> of the app<mark style="background: #FFF3A3A6;">. Global commands needed.</mark>

