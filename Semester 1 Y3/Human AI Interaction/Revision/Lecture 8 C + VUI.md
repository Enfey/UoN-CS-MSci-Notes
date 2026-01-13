# CUI
> Refers conversational user interface

Can <mark style="background: #FFF3A3A6;">refer</mark>,<mark style="background: #FFF3A3A6;"> text-based chatbots</mark>, <mark style="background: #FFF3A3A6;">voice based</mark> '<mark style="background: #FFF3A3A6;">VUIs</mark>', h<mark style="background: #FFF3A3A6;">ybrid GUI and VUI</mark> e.g., SIRI, <mark style="background: #FFF3A3A6;">many 'faces'</mark> of <mark style="background: #FFF3A3A6;">CUIs</mark>

<mark style="background: #FFF3A3A6;">Term UX</mark>; <mark style="background: #FFF3A3A6;">broad</mark>. Encompass<mark style="background: #FFF3A3A6;"> general</mark> user felt <mark style="background: #FFF3A3A6;">experience</mark> of <mark style="background: #FFF3A3A6;">interaction</mark> with <mark style="background: #FFF3A3A6;">app.</mark> 

<mark style="background: #FFF3A3A6;">Widespread adoption</mark> <mark style="background: #FFF3A3A6;">CUIs</mark>, varies, <mark style="background: #FFF3A3A6;">depending</mark> on <mark style="background: #FFF3A3A6;">nature</mark> of <mark style="background: #FFF3A3A6;">CUI</mark> + <mark style="background: #FFF3A3A6;">how CUI</mark> <mark style="background: #FFF3A3A6;">reflects values</mark> and <mark style="background: #FFF3A3A6;">needs</mark> of <mark style="background: #FFF3A3A6;">target audience</mark>. 
	<mark style="background: #FFF3A3A6;">Late 2021</mark>, <mark style="background: #FFF3A3A6;">39% adults</mark>, <mark style="background: #FFF3A3A6;">UK</mark>, <mark style="background: #FFF3A3A6;">owned</mark>,<mark style="background: #FFF3A3A6;"> used</mark>, <mark style="background: #FFF3A3A6;">voice-activated</mark> <mark style="background: #FFF3A3A6;">personal assistant,</mark> UK gov survey. 

## VUI Interaction
<mark style="background: #FFF3A3A6;">Note</mark>: number, <mark style="background: #FFF3A3A6;">problems</mark>, <mark style="background: #FFF3A3A6;">developing VUI</mark>. <mark style="background: #FFF3A3A6;">Voice</mark> = <mark style="background: #FFF3A3A6;">personal trait</mark>, <mark style="background: #FFF3A3A6;">reflects </mark><mark style="background: #FFF3A3A6;">geography</mark>,<mark style="background: #FFF3A3A6;"> sex</mark>/gender, <mark style="background: #FFF3A3A6;">sexual orientation</mark>, <mark style="background: #FFF3A3A6;">socio-economic background</mark>, more. <mark style="background: #FFF3A3A6;">Issues</mark> = <mark style="background: #FFF3A3A6;">privacy+surveillance,</mark> pertinent given intimate environment VUIs placed. 

### User input
User: Alexa, remind me, fix my clock
Alexa: I put fix my clock on your todo list

<mark style="background: #FFF3A3A6;">What happen</mark>, in between <mark style="background: #FFF3A3A6;">req + response?</mark>
### ASR
VUI, convert spoken words, into text. **<mark style="background: #FFF3A3A6;">Automatic speech recognition</mark>**

a) **<mark style="background: #FFF3A3A6;">Wakeword detection</mark>**
	<mark style="background: #FFF3A3A6;">Always listening</mark>, <mark style="background: #FFF3A3A6;">detect</mark>ed, <mark style="background: #FFF3A3A6;">starts processing</mark> speech.
b) **<mark style="background: #FFF3A3A6;">Acoustic Model-based transcription</mark>**
	<mark style="background: #FFF3A3A6;">Convert audio</mark> to <mark style="background: #FFF3A3A6;">text</mark>, <mark style="background: #FFF3A3A6;">map sound waves</mark> to <mark style="background: #FFF3A3A6;">phonemes,</mark> combine <mark style="background: #FFF3A3A6;">phonemes</mark> into <mark style="background: #FFF3A3A6;">words</mark> via <mark style="background: #FFF3A3A6;">language model.</mark>
c) **<mark style="background: #FFF3A3A6;">Confidence level</mark>**
	<mark style="background: #FFF3A3A6;">ASR,</mark> assign <mark style="background: #FFF3A3A6;">confidence score</mark>, each <mark style="background: #FFF3A3A6;">recognised word</mark>/phrase, if <mark style="background: #FFF3A3A6;">confidence low</mark>, ask<mark style="background: #FFF3A3A6;"> clarification</mark>


### NLU/NLP
Gauge intent. 

a) **<mark style="background: #FFF3A3A6;">Keyword recognition</mark>**
	<mark style="background: #FFF3A3A6;">Identify important words</mark>: 'remind', 'fix'
b) **<mark style="background: #FFF3A3A6;">Intent matching</mark>**
	<mark style="background: #FFF3A3A6;">Determine intent</mark> .
c) **<mark style="background: #FFF3A3A6;">Slot-filling</mark>**
	<mark style="background: #FFF3A3A6;">Extract details</mark>, <mark style="background: #FFF3A3A6;">acting</mark> as<mark style="background: #FFF3A3A6;"> structured data</mark>, <mark style="background: #FFF3A3A6;">system</mark> <mark style="background: #FFF3A3A6;">act upon.</mark> e.g., action and task.

### Dialogue management + TTS
<mark style="background: #FFF3A3A6;">After understanding req</mark>, <mark style="background: #FFF3A3A6;">decide</mark> how to <mark style="background: #FFF3A3A6;">response</mark>, must <mark style="background: #FFF3A3A6;">decide, if reminder</mark>, <mark style="background: #FFF3A3A6;">successfully added</mark>, <mark style="background: #FFF3A3A6;">or</mark> <mark style="background: #FFF3A3A6;">more info needed</mark>. <mark style="background: #FFF3A3A6;">TTS convert</mark>, Alexa's <mark style="background: #FFF3A3A6;">response</mark>, into <mark style="background: #FFF3A3A6;">speech.</mark> 

## Socio-economic challenges
<mark style="background: #FFF3A3A6;">7k+</mark> <mark style="background: #FFF3A3A6;">active spoken languages</mark>. <mark style="background: #FFF3A3A6;">91 10m+ speakers</mark>. <mark style="background: #FFF3A3A6;">Late 2022</mark>, <mark style="background: #FFF3A3A6;">Alexa</mark>, <mark style="background: #FFF3A3A6;">supports 8 langs</mark>, <mark style="background: #FFF3A3A6;">dialects</mark> in <mark style="background: #FFF3A3A6;">only 3</mark>. VUIs struggle more, certain dialects, accuracy lower for higher pitched voices (<mark style="background: #FFF3A3A6;">gender+age bias</mark>).


<mark style="background: #FFF3A3A6;">Systems, prone</mark>, <mark style="background: #FFF3A3A6;">racial bias</mark>, <mark style="background: #FFF3A3A6;">ASR</mark> systems at IBM, Google, Amazon, Apple, Microsoft analysed.
	<mark style="background: #FFF3A3A6;">Tested</mark>, <mark style="background: #FFF3A3A6;">transcription</mark>, <mark style="background: #FFF3A3A6;">structured interviews</mark>, conducted, with <mark style="background: #FFF3A3A6;">42</mark> white speakers, <mark style="background: #FFF3A3A6;">73</mark> black speakers. <mark style="background: #FFF3A3A6;">All exhibited racial disparities</mark>. Average <mark style="background: #FFF3A3A6;">word error rate</mark> <mark style="background: #FFF3A3A6;">= 0.35 for black</mark>
	Average <mark style="background: #FFF3A3A6;">word error rate</mark> = <mark style="background: #FFF3A3A6;">0.19 for white</mark>
	<mark style="background: #FFF3A3A6;">WER</mark> standard <mark style="background: #FFF3A3A6;">measure</mark> of <mark style="background: #FFF3A3A6;">discrepancy</mark> <mark style="background: #FFF3A3A6;">between machine</mark>+<mark style="background: #FFF3A3A6;">human transciption</mark>(<mark style="background: #FFF3A3A6;">based on</mark> <mark style="background: #FFF3A3A6;">substitutions</mark>, insertions, deletions)
	<mark style="background: #FFF3A3A6;">Disparities</mark> = <mark style="background: #FFF3A3A6;">underlying acoustic models</mark> <mark style="background: #FFF3A3A6;">used by ASR systems</mark>. 

<mark style="background: #FFF3A3A6;">Sociphonetics,</mark> field of <mark style="background: #FFF3A3A6;">linguistics,</mark> <mark style="background: #FFF3A3A6;">combines phonetics</mark>(study and classification of speech) and <mark style="background: #FFF3A3A6;">sociolingustics</mark>, determine, social factors (identity, class), influence sounds of language, how sounds are learned.
<mark style="background: #FFF3A3A6;">Phonetics</mark> = <mark style="background: #FFF3A3A6;">study of production</mark> and <mark style="background: #FFF3A3A6;">perception of spoken language</mark>. <mark style="background: #FFF3A3A6;">VUIs synthesised speech</mark>, represents<mark style="background: #FFF3A3A6;"> homogenous, mainstream accent</mark> - <mark style="background: #FFF3A3A6;">lack of diversity</mark>. One <mark style="background: #FFF3A3A6;">can ask</mark>, <mark style="background: #FFF3A3A6;">why voice assistants</mark>, <mark style="background: #FFF3A3A6;">given female voice</mark> = user trials. One <mark style="background: #FFF3A3A6;">must ask</mark>, <mark style="background: #FFF3A3A6;">whether trial</mark>, <mark style="background: #FFF3A3A6;">sufficiently represent</mark>, <mark style="background: #FFF3A3A6;">user base</mark>, <mark style="background: #FFF3A3A6;">embedding gender stereotypes</mark>.










