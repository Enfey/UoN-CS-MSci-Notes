> Process, measuring, assessing, AI model performance, ensuring reliable, accurate, safe.

# System Centred Evaluation
## Why evaluate
- Determine, whether AI system, functions, as desired
- Measures performance, evaluation permit optimisation of models, detection of weakness, compare models meaningfully.

## Evaluating classifiers
![](Pasted%20image%2020260111223955.png)
Key metrics can be derived from these. 
**Precision**
	 TP/TP+FP
	 Measure correctness of positive predictions
**Recall**
	TP/FP+FN
	Measures ability to capture all positives.
**Accuracy**
	(Correct predictions)/(All predictions)
	Sometimes misleading in imbalanced datasets, may be very good at predicting one but not the other
**F1 Score**
	Harmonic mean of precision and recall
	 $\frac{TP}{TP+ \frac{1}{2}(FP+FN)}$

### Evaluating Classifiers - Experimental design
#### Train/Test paradigm
- Dataset, 1k docs
- 60/20/20 train, validation, test split
	- Training+validation(mostly validation), tune hyperparams
	- Testing used for final evaluation, report perf.
	- Problem? How to select best hyperparameters
#### Cross-validation
- Train/test with rotation
- 1k docs
- 80/20 train/test split
	- 800 used for 4 fold rotation, each fold, becomes validation fold once.
	- Partition 800 into 4 partitions
	- Partition 1 used for validation, 2,3,4 for training -> perf 1
	- Partition 2 used for validation, 1,3,4 for training -> perf 2
- Produce stable estimate e.g., mean, standard deviation, enables hypothesis testing, good for smaller datasets as exhaustive. 
- Cannot include test set, rotation, become biased, hyperparams overfit, **optimistically biased**
- **Nested cross validation fixes**
#### Nested Cross-Validation
- 2 loops
	- Outer loop = cross validation only, used for final, unbiased eval on test set
	- Inner loop, hyperparam tuning on validation set.
- Example:
	- Outer iteration 1:
		- Test = Fold 1
		- Training pool = Folds 2, 3, 4, 5
	- Inside training pool, perform standard cross-validation, select hyper params that achieve best mean inner-cv score.
		- Inner CV Validation = Fold2, Train = F3+F4+F5
		- Inner CV Validation = Fold3, Train = F2+F4+F5
		- Inner CV Validation = Fold4, Train = F2+F3+F5
		- Inner CV Validation = Fold5, Train = F2+F3+F4
	- Once best hyperparams chosen, evaluate on outer test fold (F1), can do for multiple folds, evaluate performance on unseen folds rigorously.
	- Time consuming, computationally expensive, but does give best idea of generalisation of hyperparams to unseen data. 

## Evaluating QA systems
- Is top answer correct answer?
- Comes down, classification accuracy
	- $\frac{correctpredictions}{allpredictions}$
- If not, how far away is right answer.
	- Info retrieval eval
	- Mean reciprocal rank
		$$MRR = \frac{1}{|Q|} \cdot \sum^{|Q|}_{i=1} \frac{1}{rank_{i}}$$
	- Where $rank_i$ is position of **first** correct answer for question $i$ 
	- Suppose, 4 questions, system returns results where correct answer appears at different positions.
		 ![](Pasted%20image%2020260111225321.png)
	- RR = 1, 0.25, 0.5, 0 respectively, = 1/4(1+0.25+0.5+0).

## Evaluating NLG systems
### Intrinsic Evaluation vs Extrinsic evaluation
- **Intrinsic Evaluation**
	Eval properties of generated text
	- Fairness/bias eval
	- Robustness testing
	- Automated eval
	- Human eval
	Mechanical, precise, seek evaluate based on this precision typically.
- **Extrinisic Evaluation**
	Eval, impact, on downstream tasks, rather than direct attention to component level validity. User centred.

#### Automated Methods
- N-gram based metrics
	- **BLEU score**
		- Originally, designed, machine translation
		- Measures, how much of generated text, appears, in reference text(s)
		- Precision-oriented
		- Compute, n-gram precision between the 2, apply penalty to overly short output, balance, produced score between 0 and 1
		- For QA, ref text = correct answer. 
	- **ROUGE**
		- Designed for summarisation
		- Measure how much, of ref text, appears, in generated text
		- Recall oriented
		- QA, ref text = potential correct answers.
#### Modern methods
- **BERTScore**
	- Evolution from ngram
	- Use word embeddings, compare generated answer vs expected, token-level cosine similarity
	- Compute embeddings, each token, for each token, find most similar token in other sequence, compute precision, recall, compute f1 score.
	- Accounts, semantic variation, rather than, string matching
- **LLM-as-a-judge**
	- Modern eval, LLM used, eval NLG output, given criteria e.g., fluency
	- Eval receive system output, input prompt, explicit eval criteria, scores/ranks output accordingly.

## Drawbacks of System Centred Evaluation
- **Assume single truth**
	- Many NLP tasks, no one correct label
	- E.g., many valid paraphrases, multiple reasonable summaries
	- If only look for one gold truth, valid outputs, may be incorrectly penalised where semantics not accounted for.
- **Semantic equivalence**
	- Two outputs, same meaning, lexical difference
	- BLEU, ROUGE, accuracy etc, treat as mismatches
	- String matching metrics, cannot eval, meaning
- **Subjectivity**
	- Human annotators, may disagree due to:
		- Cultural differences
		- Contextual interpretations
		- Ambiguities
		- Low agreement between annotators, leads, unreliable evaluation metrics/criteria.
### Mitigation (human)
- **Before labelling**
	- Create, codebook, clear, annotation guidelines
- **During labelling**
	- Multiple annotators, diff perspectives, go through data
	- Pilot phase - analyse discrepancies, refine, repeat
- **After labelling**
	- Measure IAA(inter-annotator agreement(Krippendorff's Alpha))
	- If low, look more closely at data
	- Check ambiguity/bad data
## Other forms of (general) system eval
- **Fairness evaluation**
	- Check whether system, perform equally well, across demographic/social group
	- 1. **Check accuracy** across groups
	- 2. **Bias probing** using templates
		- Only works on complex NLG systems
- **Robustness evaluation/testing**
	- Check perf, when inputs, change slightly in ways, should not, change output.
	- Perturbations:
		- Typos
		- Paraphrases
		- Filler words
		- Reordered sentences
	- Ensure stability
	- May enact **Adversarial testing**
		- Measure perf against inputs, designed, break model
		- Measure worst case perf
		- E.g., add irrelevant sentences, confuse emotion classifier.


# User evaluations
## Evaluating NLG systems
### Task-based eval
- Measure impact, types of generated text, end users
- Involve, **domain, specific techniques.**
- E.g., system generates, summaries, medical data, eval by giving to doctors, assessing whether summaries, help doctors.
- Some **quant measures** e.g., decision accuracy, time to complete tasks
- **Eval multiple criteria:**
	- Is communicative goal fulfilled?
	- Is lang generated fluent/appropriate?
- **User studies**
	- Look at output, grade on criteria
	- Look at multiple outputs, choose best.

#### Best practices
1. **Determine target audience**
2. **Understand needs**
3. **Identify relevant work**
4. **Set clear goals**
5. **Validate study design**
6. **Select sample materials**
7. **Ethics approval**
8. **Experiment logs kept**
9. **Document methodology**
10. **Report results transparently**


### Evaluating chatbots - questionnaires
- PPTs recruited, based, arbitrary, criteria
- Made use chatbot, given amount of time, either unsupervised, or supervised
- PPTs fill out questionnaire, if only care about chatbot, do after, if care about effect of chatbot, do before and after
- Chatbot usability questionnaire
	- Based on SUS(system usability scale)
	- Odd-numbered questions -> positive usability
	- Even-numbered questions -> negative usability
	- Scored out of 100.
### Evaluating chatbots - perceptions
- May want measure, how people perceive chatbot
	- Healthcare chatbot should not perceived as funny
- Godspeed q'aire
	- **Anthropomorphism**
	- **Animacy**
	- **Likeability**
	- **Perceived intelligence**
	- **Perceived safety**
	- Can be applied to chatbots


### Other forms evaluation - qualitative approaches
- Can use richer qualitative methods, explore in depth, often when have small samples.
- **Think-aloud protocols**
	- Ppts, verbalise thoughts, confusion, as use chatbot, analyse the text
- **Semi-structured interviews**
	- Post study, flexible script, explore perception, attitudes etc.
- **Direct or indirect observation**
	- Observe ppt using chatbot, draw conclusions. 