, Lab focuses on using BoW representation in order to build a **classifier**, Make use of SciKit-Learn to do this. 

## Model Building
First step is to build our bag-of-word representation, by applying the CountVectoriser and TfidfTransformer to our training data. 
```python
import os
from sklearn.model_selection import train_test_split
from nltk.corpus import stopwords
from sklearn.feature_extraction.text import CountVectorizer
from sklearn.feature_extraction.text import Tfidftransformer

label_dir = {
	"positive": "data/positive"
	"negative": "data/negative"
}

data = []
labels = []

for label in label_dir.keys()
	for file in os.listdir(label_dir[label]):
		filepath = label_dir[label] + os.sep + file
		with open(filepath, encoding='utf8', 
		errors='ignore', mode = 'r') as review:
			content = review.read()
			data.append(content)
			labels.append(label)
			
X_Train, X_test, y_train, y_test = train_test_split(data, labels, stratify = labels, test_size = 0.25, random_state = 42)

count_vect = CountVectorizer(stopwords = stopwords.words('english'))
X_train_counts = count_vect.fit_transform(X_train)

tfidf_transformer = TfidfTransformer (use_idf=True,sublinear_tf=True).fit(X_train_counts)

X_train_tf = tfidf_transformer.transform(X_train_counts)

classifier = LogisticRegression(random_state=0).fit(X_train_tf , y_train)
```
Can use other classifiers such as multilayer perceptron classifier, Kneighbours classifier etc. 
## Model Evaluation
Building a classification model is actually rather straightforard. The difficult part of the whole process is making sure its doing what we want it to do, and measuring how well it is doing it. 
### Train/Test paradigm
A classification model should be trained with a specific dataset(train), and tested using another completely different data set(test). This is to ensure the model we are training is learning generalisable knowledge i.e., it can apply to unseen instances. 
```python
import os
from sklearn.model_selection import train_test_split

label_dir = {
	"positive": "data/positive"
	"negative": "data/negative"
}

data = []
labels = []

for label in label_dir.keys()
	for file in os.listdir(label_dir[label]):
		filepath = label_dir[label] + os.sep + file
		with open(filepath, encoding='utf8', 
		errors='ignore', mode = 'r') as review:
			content = review.read()
			data.append(content)
			labels.append(label)
			
X_Train, X_test, y_train, y_test = train_test_split(data, labels, stratify = labels, test_size = 0.25, random_state = 42)
```
Was detailed above, iterates through positive and negative folders according to labels, and reads the content, and places the content and label in individual lists. 

`X_train` refers to the data(text) of the training set, `y_train` to the labels of the training set, and same for `X_test` and `y_test.

The `train_test_split` method takes as input the data itself(in the form of an iterable data structure), the corresponding labels(a list of the same size), and the following arguments:
- `test_size` refers to the proportion of samples that will be kept for testing.
- `stratify` refers to the process of keeping the proportion of classes the same in training and testing
- `random_state` refers to the random seed, if not provided, the randomisation will be different every time the data is split and therefore have slightly different results. 

Stratified sampling solves the distribution matching problem, one major issue we could face is we just randomly split everything is that the distribution of classes of the test set might not match the distribution of the training set. For example, if building a sentiment classifier, the natural distribution of $[positive, neutral, negative]$ documents may look like $20\%, 70\%, 10\%$. By splitting data into training and test data sets, may end up with training set with a distribution that looks like $29\%, 70\%, 1\%$. Stratified sampling solves tis by constraining the sampling process to make sure both datasets have a class distribution that more or less matches the original distribution. 

### More Advanced Methods
The train/test way of evaluating models is great in many aspects; it's fast(training and testing once), easy to understand, and assuming selection of samples is random and from a large enough dataset, it can be fairly accurate. However, it has some major disadvantages which can be solved through slightly more advanced methods. 
#### K-Fold cross-validation
Split the dataset into $k$ roughly equal-sized subsets(folds)
The model is trained on $k-1$ folds and validated on the remaining fold, a process that is repeated $k$ times, with each fold used exactly once as the validation set. The final performance metric is the average of the scores from all $k$ iterations which provide an overall estimate of model accuracy for unseen data. 
#### Bootstrapping
Statistical resampling technique that uses sampling with replacement(put sampled items back in dataset before the next draw) to create multiple datasets from an original one, used to improve model stability and accuracy. 

Randomly sample with replacement from the original dataset to create a bootstrap sample of the same size. Compute the statistic/train the model on this sample. This is repeated $B$ times, and use the distribution of these results as needed. 
### Performance Metrics
Now that we have trained our model with the training data and training labels, we can see how it performs on unobserved data. We import and use the `accuracy_score` and `f1_score` methods from `sklearn.metrics` to measure model performance. 

```python
import os
from sklearn.model_selection import train_test_split
from nltk.corpus import stopwords
from sklearn.feature_extraction.text import CountVectorizer
from sklearn.feature_extraction.text import Tfidftransformer

label_dir = {
	"positive": "data/positive"
	"negative": "data/negative"
}

data = []
labels = []

for label in label_dir.keys()
	for file in os.listdir(label_dir[label]):
		filepath = label_dir[label] + os.sep + file
		with open(filepath, encoding='utf8', 
		errors='ignore', mode = 'r') as review:
			content = review.read()
			data.append(content)
			labels.append(label)
			
X_Train, X_test, y_train, y_test = train_test_split(data, labels, stratify = labels, test_size = 0.25, random_state = 42)

count_vect = CountVectorizer(stopwords = stopwords.words('english'))

X_train_counts = count_vect.fit_transform(X_train)

tfidf_transformer = TfidfTransformer (use_idf=True,sublinear_tf=True).fit(X_train_counts)

X_train_tf = tfidf_transformer.transform(X_train_counts)

classifier = LogisticRegression(random_state=0).fit(X_train_tf , y_train)

X_new_counts = count_vect.transform(X_test)
X_new_tfidf = tfidf_transformer.transform(X_new_counts)

predicted = clf.predict(X_new_tf_idf)
print(confusion_matrix(y_test, predicted))
print(accuracy_score(y_test, predicted))
print(f1_score(y_test, predicted, pos_label='positive'))
```
#### Confusion Matrix
A table that visualises the performance of a classification model, showing how many instances were correctly and incorrectly classified. It breaks down correct predictions into True Positives and True Negatives, and incorrect ones into False Positives and False negatives
![[Pasted image 20251105165801.png]]
Would be beneficial to have a single number to place on model correctness rather than just tell us how many incorrect samples have predicted, so can compare with other models more easily.

#### Accuracy
Accuracy is rather straightforward
$$\frac{\text{number of correct predictions}}{\text{number of predictions}}$$
Or, put in the terms defined above:
$$\frac{\text{true positives + true negatives}}{\text{true positives+true negatives+false positives+false negatives}}$$
The main strength of accuracy is that it is representative of a quantity that we care about: "if I release this model, how many times will it perform correctly, and how many times will it fail"
#### F-1 Score
Common way to put single number on the performance of a model. 
$$F_1Score = \frac{\text{true positives}}{\text{true positives} + \frac{1}{2}(false \ positives + false \ 
negatives)}$$
It is a balanced case of the general F-Score. Punishes models very badly if they ignore one class in favour of another. 



### Visualising experiment results

## Model Deployment
#### Saving and Loading a model
Joblib

#### Making new predictions




