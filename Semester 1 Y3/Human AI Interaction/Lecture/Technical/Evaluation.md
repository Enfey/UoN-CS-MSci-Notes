> Process of measuring and assessing how well an AI model performs and behaves, ensuring it is reliable and accurate, and therefore safe to deploy. 

# System Centred Evaluation
## Why Evaluate? 
- Determine whether an AI system functions as desired, and how well it does so
- Measures performance, evaluation insights permit optimisation of models, detection of weaknesses, and a way to compare models meaningfully.

## Evaluating Classifiers
![[Pasted image 20251119181842.png]]
Key metrics can be derived from these:
- **Precision** 
	- $TP / (TP + FP)$
	- Measures correctness of positive predictions.
- **Recall**
	- $TP / (FP + FN)$
	- Measures ability to capture all positives
- **Accuracy** 
	- $(Correct predictions) / (All predictions)$
	- Sometimes misleading in imbalanced datasets - may be very good at predicting one but not the other
- **F1 Score**
	- Harmonic mean of precision and recall
	- $\frac{TP}{TP+ \frac{1}{2}(FP+FN)}$

### Evaluating Classifiers - Experiment Design
#### Train/Test paradigm
- Dataset of 1000 documents
- 60/20/20 train/validation/test split
	- 600 training documents, 200 validation documents, 200 test documents
	- Training and validation used to tune hyperparameters
	- Testing is used for final evaluation
	- Problem? How to select best hyperparameters![[Pasted image 20251119182830.png]]

#### Cross-Validation
- Train/Test with rotation
- Dataset of 1000 documents
- 80/20 train/test split
	- 800 used for 4-fold rotation, each fold becomes validation once. 
	- Partition 800 into 4: $[200, 200, 200, 200]$
	- Use partition 1 for validation, partition 2,3,4 for training -> performance 1
	- Use partition 2 for validation, partition 1,3,4 for training -> performance 2 etc
- Produces more stable estimates(mean, standard deviation, enables hypothesis testing), good for smaller datasets where this exhaustive pattern will be computationally feasible. 
- If including at any point test set is included in rotation, issue. Becomes biased, cannot alter hyperparameters here as will overfit and become optimistically biased as has seen the test data before. 
- **Nested Cross Validation** fixes this

#### Nested Cross-Validation
- Creates two loops
	1. An outer loop, cross-validation only used for final, unbiased evaluation on test set. 
	2. An inner loop, used for hyperparameter tuning on the validation set.
- These are kept strictly separate.
- For example: 
	Outer Iteration 1:
		Test = Fold 1
		Training Pool = Folds 2,3,4,5
	Inside the training pool, perform standard cross-validation, to select hyperparameters that achieve the best mean inner-CV score.
		Inner CV Validation = Fold2, Train = F3+F4+F5
		Inner CV Validation = Fold3, Train = F2+F4+F5
		Inner CV Validation = Fold4, Train = F2+F3+F5
		Inner CV Validation = Fold5, Train = F2+F3+F4
	Once best hyperparameters chosen, train a fresh model on all inner-loop training folds combined using the selected hyper parameters, evaluating on the outer test fold (F1).
	Can do this for multiple folds, evaluating performance on unseen fold. 
- More time consuming and computationally expensive
- But does use data more efficiently than plain cross-validation, and gives the best idea of generalisation of hyperparameters to unseen data - model robustness and performance estimation indicated clearly. 


## Evaluating QA Systems
- Is the top answer the correct answer?
- Comes down to classification accuracy
	- $\frac{correctpredictions}{allpredictions}$
- If not, how far away is the right answer?
	- Information retrieval style evaluation
	- Mean Reciprocal Rank
		$$MRR = \frac{1}{|Q|} \cdot \sum^{|Q|}_{i=1} \frac{1}{rank_{i}}$$
	- Where $rank_i$ is the rank position of the first correct answer for question $i$.
	 ![[Pasted image 20251119185851.png]]
## Evaluating NLG systems
### Intrinsic Evaluation vs Extrinsic Evaluation
- **Intrinsic Evaluation** - evaluation of the properties of the generated text. 
	- Fairness/bias evaluation
	- Robustness/adversarial testing
	- Automated evaluations
	- Human evaluations
	Often mechanical and precise, and seeks to evaluate based on this precision typically.
- **Extrinsic Evaluation** - evaluation of the impact on downstream tasks rather than directing attention to component level validity and reliability. Often warrants user-centred evaluations. 
### Automated Methods
- N-gram based metrics
	- **BLEU** score
		- Originally designed for machine translation, now used for any textual generation task. 
		- Measures how much of the generated text appears in the reference text(s)
		- Precision oriented
		- Computes n-gram precision and applies penalty to overly short outputs to balance produced score. Produce score between 0 and 1.
		- For question answering, reference text = correct answer
	- **ROUGE**
		- Designed for summarisation, later used in other NLG tasks
		- Measures how much of the reference text appears in the generated text
		- Recall oriented
		- For question answering, reference text = potential correct answers.
### Modern Methods
- **BERTScore**
	- Evolution from ngram-based metrics. 
	- Uses word embeddings to compare generated answer and expected, computing token level cosine similarity.
	- Compute embeddings for each token, for each token, find most similar token in other sequence, compute precision and recall, and then compute f1 score.
	- Accounts for semantic variations rather than just pure string matching.
- **LLM-as-a-judge**
	- Modern evaluation method where a large language model is used to evaluate NLG output based on given criteria e.g., fluency.
	- Evaluator receives system output and input prompt, explicit evaluation criteria, and scores/ranks the output according to the criteria. 

## Drawbacks of System Centred Evaluation
- **Assumes single 'gold' truth**
	- Many NLP tasks do not have one correct label
	- For example:
		- Many valid paraphrases
		- Multiple reasonable summaries
		- Emotion labels depend on cultural interpretation
	- If only looking for one 'gold' truth, valid outputs may be incorrectly penalised
- **Semantic equivalence**
	- Two outputs may have the same meaning, but differ lexically
	- BLEU, ROUGE, accuracy etc may treat these as mismatches
	- String matching metrics therefore cannot evaluate meaning
- **Subjectivity**
	- Human annotators may disagree due to:
		- Cultural differences
		- Contextual interpretations
		- Ambiguous cases
		- Low agreement between annotators leads to unreliable evaluation metrics and a potentially untrustworthy 'ground truth'
### Mitigation (human)
- **Before labelling**
	- Create a codebook with clear annotation guidelines
- **During labelling**
	- Multiple annotators with different perspectives, go through data iteratively
	- Pilot phase - analysis of discrepancies, refinement, repeat
- **After labelling**
	- Measure IAA (inter-annotator agreement(Krippendorff's alpha))
	- If low, need to look more closely at the data
	- Check for ambiguity or bad data

### Other forms of (general) system evaluation
- **Fairness Evaluation**
	- Check whether system performs equally well across demographic or social groups
	1. Check Accuracy Across groups
		Does an emotion classifier perform worse on Scottish English or American english?
	2. Bias probing using templates
		For example: "Fill in the blank: People from the north are known to be $[?]$"
		The blank reveals whether the model shows biased stereotypes, but really only works on complex NLG systems that generate free text. 
- **Robustness testing**
	- Check performance when inputs change slightly in ways that should not change the output
	- Perturbations include:
		- Typos
		- Paraphrases
		- Filler words
		- Reordered sentences
	- The goal is to ensure stability
	- Under this scheme, one may enact **Adversarial Testing**:
		- Measure system performance against inputs designed to break the model
		- Measure the worst case performance
		- E.g., adding irrelevant sentences to confuse an emotion classifier.
# User Evaluations
## Evaluating NLG systems
### Task Based Evaluations
- Measures the impact of generated texts on end users
- Typically involves domain-specific techniques
- For example, a system that generates summaries of medical data can be evaluated by giving these summaries to doctors, and assessing whether the summaries help doctors make *better* decisions
- Some quantitative measures e.g., decision accuracy, time to complete tasks, time to complete tasks
- Must evaluate multiple criteria:
	- Is the communicative goal fulfilled?
	- Is the language generated fluent/readable/appropriate?
- User studies 
	- Look at output and grade on criteria
	- Look at multiple outputs and choose best
- There are automated approaches
	- Often oversimplify the problem, a lot of diversity in how NLG is evaluated
#### Best Practices
1. Determine target audience
2. Understand user needs
3. Identify relevant work
4. Set clear goals
5. Validate study design
6. Select sample materials
7. Obtain ethics approval
8. Keep experiment logs
9. Provide clear methodology documentation
10. Report results transparently
### Evaluating chatbots - questionnaires
- Participants recruited based on some arbitrary criteria
- Participants mde to use chatbot for given amount of time, either supervised or unsupervised
- Participants then made to fill a questionnaire survey about their experience of using the chatbot
- If we only care about the chatbot, can be done after
- If the chatbot to is meant to enact an effect on the participant, there is a questionnaire before and after the experiment.
- Chatbot usability questionnaire (Holmes 2019):
	- Based on SUS (System usability scale)
	- Odd-numbered questions -> positive usability
	- Even-numbered questions -> negative usability
	- Scored out of 100
### Evaluating chatbots - perceptions
- May want to measure how people perceive your chatbot
	- Healthcare chatbot should not be perceived as funny, unlike a companion chatbot
- The Godspeed questionnaire measures:
	- Anthropomorphism
	- Animacy
	- Likeability
	- Perceived intelligence
	- Perceived safety
	- And can be applied to chatbots (as used by Yan, Fischer & Clos, 2025)

### Other forms of evaluation - qualitative approaches
- Can use richer qualitative methods to explore in depth, often the case when have small samples
- **Think-aloud protocols** - participants verbalise their thoughts and points of confusion as they use the chatbot, and you analyse the text.
- **Semi-structured interviews** - post-study interviews with a flexible script to explore perceptions, attitudes, and/or specific incidents during experiments
- **Direct or indirect observation** - observing the participant using the chatbot and drawing conclusions from it .


