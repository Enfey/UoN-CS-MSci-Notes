> <mark style="background: #FFF3A3A6;">Process</mark>,<mark style="background: #FFF3A3A6;"> measuring</mark>, <mark style="background: #FFF3A3A6;">assessing</mark>, <mark style="background: #FFF3A3A6;">AI model performance</mark>, ensuring <mark style="background: #FFF3A3A6;">reliable</mark>, <mark style="background: #FFF3A3A6;">accurate</mark>, <mark style="background: #FFF3A3A6;">safe</mark>.

# System Centred Evaluation
## Why evaluate
- <mark style="background: #FFF3A3A6;">Determine</mark>, whether <mark style="background: #FFF3A3A6;">AI system</mark>, <mark style="background: #FFF3A3A6;">functions</mark>, as <mark style="background: #FFF3A3A6;">desired</mark>
- <mark style="background: #FFF3A3A6;">Measures performance</mark>, <mark style="background: #FFF3A3A6;">evaluation</mark> <mark style="background: #FFF3A3A6;">permit</mark> <mark style="background: #FFF3A3A6;">optimisation </mark>of <mark style="background: #FFF3A3A6;">models</mark>, <mark style="background: #FFF3A3A6;">detection</mark> of <mark style="background: #FFF3A3A6;">weakness,</mark> <mark style="background: #FFF3A3A6;">compare models </mark><mark style="background: #FFF3A3A6;">meaningfully</mark>.

## Evaluating classifiers
![](Pasted%20image%2020260111223955.png)
<mark style="background: #FFF3A3A6;">Key metrics</mark> can be <mark style="background: #FFF3A3A6;">derived</mark> from these. 
**Precision**
	<mark style="background: #FFF3A3A6;"> TP/TP+FP</mark>
	 <mark style="background: #FFF3A3A6;">Measure correctness</mark> of <mark style="background: #FFF3A3A6;">positive predictions</mark>
**Recall**
	<mark style="background: #FFF3A3A6;">TP/FP+FN</mark>
	<mark style="background: #FFF3A3A6;">Measures ability</mark> to<mark style="background: #FFF3A3A6;"> capture</mark> <mark style="background: #FFF3A3A6;">all positives.</mark>
**Accuracy**
	<mark style="background: #FFF3A3A6;">(Correct predictions)/(All predictions)</mark>
	<mark style="background: #FFF3A3A6;">Sometimes misleading</mark> in imbalanced datasets, may be very good at predicting one but not the other
**F1 Score**
	<mark style="background: #FFF3A3A6;">Harmonic mean</mark> of<mark style="background: #FFF3A3A6;"> precision</mark> and<mark style="background: #FFF3A3A6;"> recall</mark>
	 $\frac{TP}{TP+ \frac{1}{2}(FP+FN)}$

### Evaluating Classifiers - Experimental design
#### Train/Test paradigm
- Dataset, 1k docs
- <mark style="background: #FFF3A3A6;">60/20/20 train</mark>, <mark style="background: #FFF3A3A6;">validation</mark>, <mark style="background: #FFF3A3A6;">test split</mark>
	- <mark style="background: #FFF3A3A6;">Training+validation</mark>(mostly validation),<mark style="background: #FFF3A3A6;"> tune hyperparams</mark>
	- <mark style="background: #FFF3A3A6;">Testing used</mark> for<mark style="background: #FFF3A3A6;"> final evaluation</mark>, <mark style="background: #FFF3A3A6;">report perf.</mark>
	- Problem? <mark style="background: #FFF3A3A6;">How t</mark>o <mark style="background: #FFF3A3A6;">select</mark> <mark style="background: #FFF3A3A6;">best hyperparameters</mark>
#### Cross-validation
- <mark style="background: #FFF3A3A6;">Train/test</mark> with <mark style="background: #FFF3A3A6;">rotation</mark>
- <mark style="background: #FFF3A3A6;"> 1k docs</mark>
- <mark style="background: #FFF3A3A6;">80/20 train/test split</mark>
	- <mark style="background: #FFF3A3A6;">800</mark> <mark style="background: #FFF3A3A6;">used for 4 fold rotation</mark>, <mark style="background: #FFF3A3A6;">each fold</mark>, becomes <mark style="background: #FFF3A3A6;">validation fold once.</mark>
	- Partition 800 into 4 partitions
	- Partition 1 used for validation, 2,3,4 for training -> perf 1
	- Partition 2 used for validation, 1,3,4 for training -> perf 2
- <mark style="background: #FFF3A3A6;">Produce stable estimate</mark> e.g., mean, standard deviation, enables hypothesis testing, good for smaller datasets as exhaustive. 
- <mark style="background: #FFF3A3A6;">Cannot include test set</mark>, <mark style="background: #FFF3A3A6;">rotation</mark>, become biased, <mark style="background: #FFF3A3A6;">hyperparams overfit</mark>, **<mark style="background: #FFF3A3A6;">optimistically biased</mark>**
- **Nested cross validation fixes**
#### Nested Cross-Validation
- 2 loops
	- <mark style="background: #FFF3A3A6;">Outer loop</mark> = <mark style="background: #FFF3A3A6;">cross validation only</mark>, used for <mark style="background: #FFF3A3A6;">final, unbiased eval</mark> on test set
	- Inner loop, hyperparam tuning on validation set.
- Example:
	- <mark style="background: #FFF3A3A6;">Outer iteration 1</mark>:
		- <mark style="background: #FFF3A3A6;">Test</mark> = <mark style="background: #FFF3A3A6;">Fold 1</mark>
		- <mark style="background: #FFF3A3A6;">Training pool</mark> = Folds 2, 3, 4, 5
	- <mark style="background: #FFF3A3A6;">Inside training poo</mark>l, perform<mark style="background: #FFF3A3A6;"> standard cross-validation</mark>, <mark style="background: #FFF3A3A6;">select hyper params</mark> that <mark style="background: #FFF3A3A6;">achieve</mark> <mark style="background: #FFF3A3A6;">best mean inner-cv score</mark>.
		- Inner CV Validation = Fold2, Train = F3+F4+F5
		- Inner CV Validation = Fold3, Train = F2+F4+F5
		- Inner CV Validation = Fold4, Train = F2+F3+F5
		- Inner CV Validation = Fold5, Train = F2+F3+F4
	- Once <mark style="background: #FFF3A3A6;">best hyperparams chosen,</mark><mark style="background: #FFF3A3A6;"> evaluate</mark> on<mark style="background: #FFF3A3A6;"> outer test fold</mark> (F1), can <mark style="background: #FFF3A3A6;">do for multiple folds,</mark> <mark style="background: #FFF3A3A6;">evaluate performance</mark> on <mark style="background: #FFF3A3A6;">unseen folds</mark> rigorously.
	- <mark style="background: #FFF3A3A6;">Time consuming,</mark> computationally expensive, but does give <mark style="background: #FFF3A3A6;">best idea of generalisation of hyperparams</mark> to unseen data. 

## Evaluating QA systems
- Is <mark style="background: #FFF3A3A6;">top answer correct answer</mark>?
- <mark style="background: #FFF3A3A6;">Comes down, classification accuracy</mark>
	- $\frac{correctpredictions}{allpredictions}$
- <mark style="background: #FFF3A3A6;">If not</mark>, <mark style="background: #FFF3A3A6;">how far away</mark> is <mark style="background: #FFF3A3A6;">right answer</mark>.
	- <mark style="background: #FFF3A3A6;">Info retrieval eval</mark>
	- <mark style="background: #FFF3A3A6;">Mean reciprocal rank</mark>
		$$MRR = \frac{1}{|Q|} \cdot \sum^{|Q|}_{i=1} \frac{1}{rank_{i}}$$
	- <mark style="background: #FFF3A3A6;">Where</mark> $rank_i$ is <mark style="background: #FFF3A3A6;">position</mark> of **<mark style="background: #FFF3A3A6;">first</mark>** <mark style="background: #FFF3A3A6;">correct answer</mark> for question $i$ 
	- Suppose, <mark style="background: #FFF3A3A6;">4 questions</mark>, system <mark style="background: #FFF3A3A6;">returns results</mark> where <mark style="background: #FFF3A3A6;">correct answer</mark> <mark style="background: #FFF3A3A6;">appears</mark> at <mark style="background: #FFF3A3A6;">different positions</mark>.
		 ![](Pasted%20image%2020260111225321.png)
	- RR = 1, 0.25, 0.5, 0 respectively, = 1/4(1+0.25+0.5+0).

## Evaluating NLG systems
### Intrinsic Evaluation vs Extrinsic evaluation
- **<mark style="background: #FFF3A3A6;">Intrinsic Evaluation</mark>**
	<mark style="background: #FFF3A3A6;">Eval properties</mark> of <mark style="background: #FFF3A3A6;">generated text</mark>
	- <mark style="background: #FFF3A3A6;">Fairness</mark>/<mark style="background: #FFF3A3A6;">bias eval</mark>
	- <mark style="background: #FFF3A3A6;">Robustness testing</mark>
	- <mark style="background: #FFF3A3A6;">Automated eval</mark>
	- <mark style="background: #FFF3A3A6;">Human eval</mark>
	<mark style="background: #FFF3A3A6;">Mechanical</mark>, <mark style="background: #FFF3A3A6;">precise</mark>, seek evaluate based on this precision typically.
- **Extrinisic Evaluation**
	<mark style="background: #FFF3A3A6;">Eval</mark>, <mark style="background: #FFF3A3A6;">impact, </mark>on<mark style="background: #FFF3A3A6;"> downstream tasks</mark>, <mark style="background: #FFF3A3A6;">rather than</mark> <mark style="background: #FFF3A3A6;">direct attention</mark> to <mark style="background: #FFF3A3A6;">component level</mark> <mark style="background: #FFF3A3A6;">validity</mark>. User centred.

#### Automated Methods
- <mark style="background: #FFF3A3A6;">N-gram based metrics</mark>
	- **<mark style="background: #FFF3A3A6;">BLEU score</mark>**
		- Originally, designed, machine translation
		- <mark style="background: #FFF3A3A6;">Measures</mark>, <mark style="background: #FFF3A3A6;">how much</mark> of <mark style="background: #FFF3A3A6;">generated text</mark>, <mark style="background: #FFF3A3A6;">appears</mark>, in <mark style="background: #FFF3A3A6;">reference text</mark>(s)
		- <mark style="background: #FFF3A3A6;">Precision-oriented</mark>
		- <mark style="background: #FFF3A3A6;">Compute</mark>, <mark style="background: #FFF3A3A6;">n-gram precision</mark> <mark style="background: #FFF3A3A6;">between the 2</mark>, <mark style="background: #FFF3A3A6;">apply penalty</mark> to <mark style="background: #FFF3A3A6;">overly short output</mark>,<mark style="background: #FFF3A3A6;"> balance</mark>, produced <mark style="background: #FFF3A3A6;">score between 0 and 1</mark>
		- For <mark style="background: #FFF3A3A6;">QA</mark>, <mark style="background: #FFF3A3A6;">ref text</mark> = <mark style="background: #FFF3A3A6;">correct answer. </mark>
	- **<mark style="background: #FFF3A3A6;">ROUGE</mark>**
		- Designed for summarisation
		- <mark style="background: #FFF3A3A6;">Measure how much</mark>, of <mark style="background: #FFF3A3A6;">ref text</mark>, <mark style="background: #FFF3A3A6;">appear</mark>s, in<mark style="background: #FFF3A3A6;"> generated text</mark>
		- <mark style="background: #FFF3A3A6;">Recall oriented</mark>
		- <mark style="background: #FFF3A3A6;">QA,</mark> <mark style="background: #FFF3A3A6;">ref text</mark> =<mark style="background: #FFF3A3A6;"> potential correct answers.</mark>
#### Modern methods
- **<mark style="background: #FFF3A3A6;">BERTScore</mark>**
	- <mark style="background: #FFF3A3A6;">Evolution from ngram</mark>
	- Use <mark style="background: #FFF3A3A6;">word embeddings</mark>, compare <mark style="background: #FFF3A3A6;">generated answer</mark> vs <mark style="background: #FFF3A3A6;">expected</mark>, <mark style="background: #FFF3A3A6;">token-level cosine similarity</mark>
	- <mark style="background: #FFF3A3A6;">Compute embeddings</mark>, <mark style="background: #FFF3A3A6;">each token</mark>, for <mark style="background: #FFF3A3A6;">each token</mark><mark style="background: #FFF3A3A6;">, find most similar token</mark> in <mark style="background: #FFF3A3A6;">other sequence</mark>, <mark style="background: #FFF3A3A6;">compute precision, recall</mark>, compute <mark style="background: #FFF3A3A6;">f1 score.</mark>
	- <mark style="background: #FFF3A3A6;">Accounts</mark>, <mark style="background: #FFF3A3A6;">semantic variation,</mark> <mark style="background: #FFF3A3A6;">rather than</mark>, <mark style="background: #FFF3A3A6;">string matching</mark>
- **LLM-as-a-judge**
	- <mark style="background: #FFF3A3A6;">Modern eval</mark>, <mark style="background: #FFF3A3A6;">LLM used</mark>,<mark style="background: #FFF3A3A6;"> eval NLG output</mark>, <mark style="background: #FFF3A3A6;">given criteria e.g., fluency</mark>
	- <mark style="background: #FFF3A3A6;">Eval receive system output,</mark> <mark style="background: #FFF3A3A6;">input prompt</mark>, explicit<mark style="background: #FFF3A3A6;"> eval criteria</mark>, scores/<mark style="background: #FFF3A3A6;">ranks output</mark> accordingly.

## Drawbacks of System Centred Evaluation
- **<mark style="background: #FFF3A3A6;">Assume single truth</mark>**
	- <mark style="background: #FFF3A3A6;">Many NLP tasks</mark>, <mark style="background: #FFF3A3A6;">no one correct label</mark>
	- E.g., <mark style="background: #FFF3A3A6;">many valid paraphrases</mark>, <mark style="background: #FFF3A3A6;">multiple reasonable summaries</mark>
	- If <mark style="background: #FFF3A3A6;">only look</mark> for <mark style="background: #FFF3A3A6;">one gold truth</mark>, <mark style="background: #FFF3A3A6;">valid outputs</mark>, may be <mark style="background: #FFF3A3A6;">incorrectly penalised </mark>where semantics not accounted for.
- **<mark style="background: #FFF3A3A6;">Semantic equivalence</mark>**
	- <mark style="background: #FFF3A3A6;">Two outputs,</mark> <mark style="background: #FFF3A3A6;">same meaning</mark>, <mark style="background: #FFF3A3A6;">lexical difference</mark>
	- <mark style="background: #FFF3A3A6;">BLEU, ROUGE</mark>, <mark style="background: #FFF3A3A6;">accuracy</mark> etc, <mark style="background: #FFF3A3A6;">treat as mismatches</mark>
	- <mark style="background: #FFF3A3A6;">String matching</mark> metrics, <mark style="background: #FFF3A3A6;">cannot eval</mark>, <mark style="background: #FFF3A3A6;">meaning</mark>
- **<mark style="background: #FFF3A3A6;">Subjectivity</mark>**
	- <mark style="background: #FFF3A3A6;">Human annotators,</mark> may <mark style="background: #FFF3A3A6;">disagree due to</mark>:
		- <mark style="background: #FFF3A3A6;">Cultural differences</mark>
		- <mark style="background: #FFF3A3A6;">Contextual interpretations</mark>
		- <mark style="background: #FFF3A3A6;">Ambiguities</mark>
		- <mark style="background: #FFF3A3A6;">Low agreement between annotators</mark>, leads, unreliable evaluation metrics/criteria.
### Mitigation (human)
- **<mark style="background: #FFF3A3A6;">Before labelling</mark>**
	- <mark style="background: #FFF3A3A6;">Create, codebook</mark>, clear, <mark style="background: #FFF3A3A6;">annotation guidelines</mark>
- **<mark style="background: #FFF3A3A6;">During labelling**</mark>
	- <mark style="background: #FFF3A3A6;">Multiple annotators</mark>, <mark style="background: #FFF3A3A6;">diff perspectives</mark>, <mark style="background: #FFF3A3A6;">go through data</mark>
	- <mark style="background: #FFF3A3A6;">Pilot phase</mark> -<mark style="background: #FFF3A3A6;"> analyse discrepancies</mark>, <mark style="background: #FFF3A3A6;">refine</mark>, <mark style="background: #FFF3A3A6;">repeat</mark>
- **<mark style="background: #FFF3A3A6;">After labelling</mark>**
	- <mark style="background: #FFF3A3A6;">Measure IAA</mark>(inter-annotator agreement(K<mark style="background: #FFF3A3A6;">rippendorff's Alpha</mark>))
	- <mark style="background: #FFF3A3A6;">If low</mark>, <mark style="background: #FFF3A3A6;">look more closely at data</mark>
	- <mark style="background: #FFF3A3A6;">Check ambiguity</mark>/<mark style="background: #FFF3A3A6;">bad data</mark>
## Other forms of (general) system eval
- **<mark style="background: #FFF3A3A6;">Fairness evaluation</mark>**
	- <mark style="background: #FFF3A3A6;">Check</mark> <mark style="background: #FFF3A3A6;">whether system</mark>, <mark style="background: #FFF3A3A6;">perform equally well</mark>, across <mark style="background: #FFF3A3A6;">demographic</mark>/social group
	- 1. **<mark style="background: #FFF3A3A6;">Check accuracy</mark>** <mark style="background: #FFF3A3A6;">across groups</mark>
	- 2. **<mark style="background: #FFF3A3A6;">Bias probing</mark>** <mark style="background: #FFF3A3A6;">using templates</mark>
		- <mark style="background: #FFF3A3A6;">Only works</mark> on <mark style="background: #FFF3A3A6;">complex NLG systems</mark>
- **<mark style="background: #FFF3A3A6;">Robustness evaluation/testing</mark>**
	- <mark style="background: #FFF3A3A6;">Check perf</mark>, when <mark style="background: #FFF3A3A6;">inputs, change slightly in ways</mark>, <mark style="background: #FFF3A3A6;">should not, change output</mark>.
	- <mark style="background: #FFF3A3A6;">Perturbations</mark>:
		- <mark style="background: #FFF3A3A6;">Typos</mark>
		- <mark style="background: #FFF3A3A6;">Paraphrases</mark>
		- <mark style="background: #FFF3A3A6;">Filler words</mark>
		- <mark style="background: #FFF3A3A6;">Reordered </mark>sentences
	- Ensure <mark style="background: #FFF3A3A6;">stability</mark>
	- May <mark style="background: #FFF3A3A6;">enact</mark> **<mark style="background: #FFF3A3A6;">Adversarial testing</mark>**
		- Measure perf against inputs, designed, break model
		- Measure worst case perf
		- E.g., add <mark style="background: #FFF3A3A6;">irrelevant sentences,</mark> <mark style="background: #FFF3A3A6;">confuse</mark> <mark style="background: #FFF3A3A6;">emotion classifier.</mark>


# User evaluations
## Evaluating NLG systems
### <mark style="background: #FFF3A3A6;">Task-based eval</mark>
- <mark style="background: #FFF3A3A6;">Measure impact</mark>, <mark style="background: #FFF3A3A6;">types</mark> of <mark style="background: #FFF3A3A6;">generated text</mark>, <mark style="background: #FFF3A3A6;">end users</mark>
- Involve, <mark style="background: #FFF3A3A6;">**domain</mark>, <mark style="background: #FFF3A3A6;">specific</mark> <mark style="background: #FFF3A3A6;">techniques</mark>.**
- E.g., <mark style="background: #FFF3A3A6;">system generates</mark>,<mark style="background: #FFF3A3A6;"> summaries</mark>,<mark style="background: #FFF3A3A6;"> medical data</mark>, <mark style="background: #FFF3A3A6;">eval</mark> by <mark style="background: #FFF3A3A6;">giving</mark> to <mark style="background: #FFF3A3A6;">doctors,</mark><mark style="background: #FFF3A3A6;"> assessing</mark> <mark style="background: #FFF3A3A6;">whether summaries</mark>,<mark style="background: #FFF3A3A6;"> help</mark> doctors.
- <mark style="background: #FFF3A3A6;">Some</mark> **<mark style="background: #FFF3A3A6;">quant measures</mark>** e.g., <mark style="background: #FFF3A3A6;">decision accuracy</mark>, <mark style="background: #FFF3A3A6;">time to complete tasks</mark>
- **<mark style="background: #FFF3A3A6;">Eval multiple criteria</mark>:**
	- <mark style="background: #FFF3A3A6;">Is communicative goal fulfilled?</mark>
	- <mark style="background: #FFF3A3A6;">Is lang generated</mark> fluent/<mark style="background: #FFF3A3A6;">appropriate</mark>?
- **<mark style="background: #FFF3A3A6;">User studies</mark>**
	- <mark style="background: #FFF3A3A6;">Look at output,</mark> <mark style="background: #FFF3A3A6;">grade on criteria</mark>
	- <mark style="background: #FFF3A3A6;">Look at multiple outputs</mark>,<mark style="background: #FFF3A3A6;"> choose best.</mark>

#### Best practices
1. **<mark style="background: #FFF3A3A6;">Determine target audience</mark>**
2. **<mark style="background: #FFF3A3A6;">Understand needs</mark>**
3. **<mark style="background: #FFF3A3A6;">Identify relevant work</mark>**
4. **<mark style="background: #FFF3A3A6;">Set clear goals</mark>**
5. **<mark style="background: #FFF3A3A6;">Validate study design</mark>**
6. **<mark style="background: #FFF3A3A6;">Select sample materials</mark>**
7. **<mark style="background: #FFF3A3A6;">Ethics approva</mark>l**
8. **<mark style="background: #FFF3A3A6;">Experiment logs kept</mark>**
9. **<mark style="background: #FFF3A3A6;">Document methodology</mark>**
10. **<mark style="background: #FFF3A3A6;">Report results transparently</mark>**


### Evaluating chatbots - questionnaires
- <mark style="background: #FFF3A3A6;">PPTs recruited</mark>, based, arbitrary, <mark style="background: #FFF3A3A6;">criteria</mark>
- Made <mark style="background: #FFF3A3A6;">use chatbot</mark>, given <mark style="background: #FFF3A3A6;">amount of time</mark>, either <mark style="background: #FFF3A3A6;">unsupervised</mark>, or <mark style="background: #FFF3A3A6;">supervised</mark>
- PPTs <mark style="background: #FFF3A3A6;">fill out questionnaire</mark>, if only care about chatbot, do after, if care about effect of chatbot, do before and after
- <mark style="background: #FFF3A3A6;">Chatbot usability questionnaire</mark>
	- <mark style="background: #FFF3A3A6;">Based on SUS</mark>(<mark style="background: #FFF3A3A6;">system usability scale</mark>)
	- <mark style="background: #FFF3A3A6;">Odd-numbered questions</mark> -> <mark style="background: #FFF3A3A6;">positive usability</mark>
	- <mark style="background: #FFF3A3A6;">Even-numbered questions </mark>-> <mark style="background: #FFF3A3A6;">negative usability</mark>
	- <mark style="background: #FFF3A3A6;">Scored out of 100.</mark>
### Evaluating chatbots - perceptions
- May want <mark style="background: #FFF3A3A6;">measure</mark>, how<mark style="background: #FFF3A3A6;"> people</mark> <mark style="background: #FFF3A3A6;">perceive </mark><mark style="background: #FFF3A3A6;">chatbot</mark>
	- <mark style="background: #FFF3A3A6;">Healthcare chatbot</mark> should <mark style="background: #FFF3A3A6;">not</mark> perceived as <mark style="background: #FFF3A3A6;">funny</mark>
- Godspeed q'aire
	- **<mark style="background: #FFF3A3A6;">Anthropomorphism</mark>**
	- **<mark style="background: #FFF3A3A6;">Animacy</mark>**
	- **<mark style="background: #FFF3A3A6;">Likeability</mark>**
	- **Perceived <mark style="background: #FFF3A3A6;">intelligence</mark>**
	- **Perceived <mark style="background: #FFF3A3A6;">safety</mark>**
	- Can be applied to chatbots


### Other forms evaluation - qualitative approaches
- Can <mark style="background: #FFF3A3A6;">use richer qualitative methods</mark>, <mark style="background: #FFF3A3A6;">explore in depth</mark>, often when have small samples.
- **<mark style="background: #FFF3A3A6;">Think-aloud protocols</mark>**
	- <mark style="background: #FFF3A3A6;">Ppts</mark>,<mark style="background: #FFF3A3A6;"> verbalise thoughts</mark>, <mark style="background: #FFF3A3A6;">confusion</mark>, as <mark style="background: #FFF3A3A6;">use chatbot</mark>, analyse the text
- **<mark style="background: #FFF3A3A6;">Semi-structured interviews</mark>**
	- Post study, flexible script, explore perception, attitudes etc.
- **<mark style="background: #FFF3A3A6;">Direct or indirect observation</mark>**
	- Observe ppt using chatbot, draw conclusions. 