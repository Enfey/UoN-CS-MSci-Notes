# CUI
> Refers conversational user interface

Can refer, text-based chatbots, voice based 'VUIs', hybrid GUI and VUI e.g., SIRI, many 'faces' of CUIs

Term UX; broad. Encompass general user felt experience of interaction with app. 

Widespread adoption CUIs, varies, depending on nature of CUI + how CUI reflects values and needs of target audience. 
	Late 2021, 39% adults, UK, owned, used, voice-activated personal assistant, UK gov survey. 

## VUI Interaction
Note: number, problems, developing VUI. Voice = personal trait, reflects geography, sex/gender, sexual orientation, socio-economic background, more. Issues = privacy+surveillance, pertinent given intimate environment VUIs placed. 

### User input
User: Alexa, remind me, fix my clock
Alexa: I put fix my clock on your todo list

What happen, in between req + response?
### ASR
VUI, convert spoken words, into text. **Automatic speech recognition**

a) **Wakeword detection**
	Always listening, detected, starts processing speech.
b) **Acoustic Model-based transcription**
	Convert audio to text, map sound waves to phonemes, combine phonemes into words via language model.
c) **Confidence level**
	ASR, assign confidence score, each recognised word/phrase, if confidence low, ask clarification


### NLU/NLP
Gauge intent. 

a) **Keyword recognition**
	Identify important words: 'remind', 'fix'
b) **Intent matching**
	Determine intent
c) **Slot-filling**
	Extract details, acting as structured data, system act upon. e.g., action and task.

### Dialogue management + TTS
After understanding req, decide how to response, must decide, if reminder, successfully added, or more info needed. TTS convert, Alexa's response, into speech. 

## Socio-economic challenges
7k+ active spoken languages. 91 10m+ speakers. Late 2022, Alexa, supports 8 langs, dialects in only 3. VUIs struggle more, certain dialects, accuracy lower for higher pitched voices (gender+age bias).


Systems, prone, racial bias, ASR systems at IBM, Google, Amazon, Apple, Microsoft analysed.
	Tested, transcription, structured interviews, conducted, with 42 white speakers, 73 black speakers. All exhibited racial disparities. Average word error rate = 0.35 for black
	Average word error rate = 0.19 for white
	WER standard measure of discrepancy between machine+human transciption(based on substitutions, insertions, deletions)
	Disparities = underlying acoustic models used by ASR systems. 

Sociphonetics, field of linguistics, combines phonetics(study and classification of speech) and sociolingustics, determine, social factors (identity, class), influence sounds of language, how sounds are learned.
Phonetics = study of production and perception of spoken language. VUIs synthesised speech, represents homogenous, mainstream accent - lack of diversity. One can ask, why voice assistants, given female voice = user trials. One must ask, whether trial, sufficiently represent, user base, embedding gender stereotypes.










