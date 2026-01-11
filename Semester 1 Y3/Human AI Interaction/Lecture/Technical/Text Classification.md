Documents in our corpus are represented by the TF-IDF weighting of their positive and negative words yielding a representation in 2-dimensional vector space, after preprocessing and representation. The sentiment classes are positive, and negative. 
$$
\begin{array}{c}
P_{1}[3,6] \to positive  \\
P_{2}[1.5, 6] \to positive\\
P_{3}[2,4] \to positive\\
P_{4}[4.5,2] \to negative \\
P_{5}[5.5, 3] \to negative \\
P_{6}[6.5, 4.5] \to negative
\end{array}
$$
![[Pasted image 20251105094517.png]]
How can we separate these two classes, and classify documents positive or negative based on their features?

## Classifying Text
The goal is to find a rule or model which can separate classes and generalise. 

Several classifier types exist.

### Memorisation
Stores exact examples, perfect on training data and is fast, but obviously has the inability to generalise to unseen instances. 
### OneR
One derives a simple rule, e.g., if $x \geq 3$ then negative, otherwise positive. This is easily interpretable and can generalise(although very poorly.)
### Linear Classifier
One draws a line to separate classes e.g., $y = 1.5 \cdot x$. This is very fast and generalises well, though one is limited to linear separability. 
This can be represented in $n$ dimensions via $$y = \theta_{0} + \sum_{i=1}^n\theta_{i}x_{i}$$
This states that the output y is a weighted sum of the input features plus a bias term $\theta_{0}$.

Yields a plane or hyperplane in higher dimensions. 
### Nearest Neighbour Classifier
Classify based on neighbourhood. New data is classified based on closest neighbouring points. Low level of generalisation error, and speed depends on the model. 
![[Pasted image 20251105100752.png]]

### Decision Tree classifier
Splits data by thresholds, and can be efficient, however can overfit![[Pasted image 20251105100837.png]]

### Classification as generalisation
Core assumption is that our data is representative of the real world. Generalisation challenges include things such as vocabulary drift, temporal shifts in data, and geographical, linguistic, or cultural differences. We never observe the real world - only a sample of it. One must ensure that model assumptions match reality.

## Machine learning
About learning from examples - automatically discovering patterns in data that can therefore generalise to new, unseen data. In text classification, ML models learn decision boundaries that separate categories such as:
- *positive vs negative sentiment*
- *spam vs non-spam*
- *intent type*
Replaces hand-crafted rules with data-driven learning. 

### Training, testing, and generalisation
Three types of error are introduced
1. Training error - how well the model fits the training data
2. Testing error - how well the model performs on unseen testing data
3. Generalisation error - how well the model performs in the real world
The goal is a low generalisation error, and avoiding overfitting. We cannot really measure the generalisation error, but we use the training and testing error and some clever experiment design to estimate it.
- Train/test split - reserve part of the data for testing only
- Cross-Validation - repeatedly train/test on different partitions to get a stable estimate
- Bootstrapping - sample data with replacement to create multiple training/test sets
### Types of ML models for text classifications
#### Traditional ML models
These require manual feature extraction e.g., Bag of Words, TF-IDF
- **Linear models** - Logistic regression
- **Tree models** - Decision Trees
- **Probabilistic models** - Naive bayes, Bayesian Networks
- **K-Nearest Neighbour** - K-Nearest Neighbour
Representation is explicitly defined by user and fed to the model, and thus reliant on good text representation.

#### Deep Learning Models
These learn representations automatically through multiple layers:
- **Perceptron/Multi-Layer Perceptron**
- **Convolutional Neural Networks**
- **Recurrent Neural Networks**
- **Transformers**

### ML approaches
- **Supervised** - there is known ground truth about the training data i.e., each data point has a label associated
- **Unsupervised** - there is no known ground truth about the data i.e., no labels associated to data points
- **Semi-Supervised** - there is ground truth about some of the data i.e., some data points have labels, others do not.
- **Reinforcement Learning** - technique where an agent learns to make decisions through trial and error to maximise a reward signal, receives feedback in form of rewards or penalties.

### Learning and Optimisation
At the heart of ML is the learning process, where model parameters are tuned to minimise a loss function. 

The perceptron preceded modern neural networks.
$$y = f(\sum_{i}w_{i}x_{i}+b)$$
Where $f$ is a non-linear activation. 
Typical functions
Step function: $f(x) = 1 \ if \ \frac{1}{1+e^{-x}} \geq 0 \ else -1$ 
Sigmoid: $f(x) = \frac{1}{1+e^{-x}}$
Hyperbolic Tangent: $f(x) = tanh(w \cdot x + b)$ 
ReLU: $f(x) = max(0, x)$ 

Given inputs $x1...x_n$ and weights $w_1...w_n$, the model outputs $y$ and compares it to the true label to compute **loss**.

New input $P_1$ represented by 3 features

$p_1: x_1 = 0.3, x_2 = 0.3, x_3 = 0.3$
$p_1 : w_1 = 1.5, w_2 = -1.5, w_3 = 3.3, w_b = 0.3$ 

$f(1.29)$
$\frac{1}{1+e^{1.29}} = 0.78471$ using step function.
Decision boundary of is reached so $P_1$ classified as $1$.


#### Learning via gradient descent
Once the model starts making predictions, it needs to learn better weights, which is where the loss function and gradient descent come in.

The loss function $\mathcal{J}$ measures how wrong the model's predictions are, want this to be minimised. 

We iteratively update parameters $w$ over $n$ iterations using the partial derivative of the loss function with respect to the parameters.

$$w_{t+1} = w^t - \mathcal{N} \cdot \Delta_{w^t} \mathcal{J}(w^t)$$
$\mathcal{N}$ is a fixed hyperparameter that controls the learning rate/step size

Hyperparameter is set learned automatically - set before training, and control how learning happens. 

1. Choose a hyperparameter that looks reasonable and stick to it
	Assumption: it does not influence much
2. Choose a hyperparameter from the literature
	Assumption: it does matter, but there are known best practices
3. Find the best hyperpara,meter possible on own dataset. 

### Neural Networks and Backpropagation
In a neural network, have layers of neurons, each neuron performs:
$$y = f(\sum_{i}w_{i}x_{i}+b)$$
The network predicts an output, computes the error, and backpropagation  calculates how much each weight contributed to that error. 

Then it updates weights to reduce the error using the same gradient descent rule applied layer by layer. 

Multiclass classification, cannot use sigmoid output. We usually use a **softmax** function
$$softmax (z_{i}) = \frac{e^{z_{i}}} {\sum^C_{j=1} e^{z_{i}}}$$
Produces a probability distribution over all $C$ classes, the class with the highest probability is chosen as the prediction. Differentiable, so can still use backpropagation. 

### Deep learning
Same process, more layers and nonlinearities, each layer learns more abstraction representations of the input. Lower layers detect simple features like words, and higher layers detect complex patterns. Overfitting is always possible, and is expensive to train with lots of examples. There are techniques to reduce overfitting such as dropout(randomly disable neurons) and regularisation(L1/L2 penalties) and early stopping, stopping the learning process before convergence

### Regularisation
The process of finding the best trade-off between simplicity and training error is regularisation.

We want to find the best compromise between minimising training error and minimising model complexity. 

We modify the cost function to penalise complexity:
$$Cost(\beta) = Error(\beta, X, Y) + \lambda \cdot Complexity(\beta)$$
where:
	$\lambda$ : regularisation parameter controlling penalty strength
	$Complexity(\beta)$: depends on the size of the weights
Common ways of regularising linear models:
$$
\text{Complexity}(\boldsymbol{\theta}) = \lambda \sum_{i=1}^{n} \theta_i{^2}
$$
*L2* regularisation

$$\text{Complexity}(\boldsymbol{\theta}) = \lambda \sum_{i=1}^{n} |\theta_i|$$
*L1* regularisation
$$ 
\text{Complexity}(\boldsymbol{\theta}) = 
\lambda_1 \sum_{i=1}^{n} \theta_i^2 + 
\lambda_2 \sum_{i=1}^{n} |\theta_i|$$
Elastic net regularisation












