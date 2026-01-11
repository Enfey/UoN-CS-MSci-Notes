Documents in corpus, represented, TF-IDF weighting of their positive + negative words, yielding representation in 2 dimensional vector space after preprocessing and representation. Sentiment classes are positive, and negative. 

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
How can we separate these two classes + classify documents pos/neg based on features?


## Classifying Text
- Goal = find rule/model, that can separate classes, generalise

### Memorisation
Store exact samples, perfect on training data, cannot generalise.

### OneR
Derives simple rule e.g., if x > 3 then negative, otherwise positive. Easily interpretable, can generalise (poorly).

### Linear Classifier
Draws line, separate classes. Fast, generalises well, limited to linear separability. 
 $$y = \theta_{0} + \sum_{i=1}^n\theta_{i}x_{i}$$
 
This states that the output y is a weighted sum of the input features plus a bias term $\theta_{0}$. Yield line, plane, or hyperplane. 

### Nearest neighbour classifier
- Classify, based on neighbourhood, new data, classified, based on closest neighbouring points. Low level of generalisation error, speed depends on the model.
- ![[Pasted image 20251105100752.png]]
### Decision tree classifier
Split data by thresholds, can be efficient, speed depends a lot on model little space, generalise well but can overfit.
![[Pasted image 20251105100837.png]]

### Classification as generalisation
Core assumption, data is representative of real world. 
Generalisation challenge; vocabulary shifts, geographical, linguistic, cultural differences, temporal shifts in the data. Never observe the real world, only sample; must ensure model assumptions, match reality.


## Machine learning
Learning via examples, discover patterns in data, generalise to new, unseen. Text classification, learn decision boundaries to separate categories. 

### Training, testing, generalisation
Three types of error introduced
1. Training error - how well model fits training data
2. Testing error - how well model performs on unseen testing data
3. Generalisation error - how well model performs in real world
Optimise generalisation error; cannot measure it, so use other 2 and experimental design to estimate it. 
- **Train/test split** - reserve part of data for testing only
- **Cross-validation** - repeatedly train/test on diff partitions, get stable estimate
- **Bootstrapping**- sample data with replacement, create multiple train/test sets.

### Types of ML models for text classification
#### Traditional ML models
- Require manual feature extraction e.g., BOW, TF-IDF
	- Linear models - LR
	- Tree models - decision trees
	- Probabilistic models - naive bayes
	- KNN
- Representation is user defined, fed to model.
#### Deep learning models
Learn representations automatically via multiple layers; perceptron/multi-layer perceptron, CNNs, RNNs, Transformers.

### ML approaches
- **Supervised** - known ground truth about training data
- **Unsupervised** - no ground truth about the data
- **Semi-supervised** - ground truth about some of data
- **Reinforcement learning** - technique, agent learns, make decisions via trial+error to maximise reward signal, receives feedback in form of rewards/penalties. 

## Learning and optimisation
At heart of ML, learning, model parameters tuned to minimise loss function.

Perceptron, preceded modern NN.
$$y = f(\sum_{i}w_{i}x_{i}+b)$$
Where $f$ is non-linear activation.
Typical functions:
Step function: $f(x) = 1 \ if \ \frac{1}{1+e^{-x}} \geq 0 \ else -1$ 
Sigmoid: $f(x) = \frac{1}{1+e^{-x}}$
Hyperbolic Tangent: $f(x) = tanh(w \cdot x + b)$ 
ReLU: $f(x) = max(0, x)$ 

Given inputs $x_1\dots x_{n}$ and weights $w_1\dots w_{n}$ model outputs $y$ and compares to true label to compute loss. 

New input $p_1$ represented by 3 features.

$p_1: x_1 = 0.3, x_2 = 0.3, x_3 = 0.3$
$p_1 : w_1 = 1.5, w_2 = -1.5, w_3 = 3.3, w_b = 0.3$ 

f(1.29)
$\frac{1}{1+e^{1.29}} = 0.78471$ using step function.
Decision boundary reached, classified as 1. 

### Learning via gradient descent
Once model starts making predictions, needs to learn better weights, where loss function + gradient descent come in. 

Loss function $\mathcal{J}$ measures how incorrect model predictions are, minimise.

Iteratively update parameters $w$ over $n$ iterations using the partial derivate of loss function with respect to the parameters.


$$w_{t+1} = w^t - \mathcal{N} \cdot \Delta_{w^t} \mathcal{J}(w^t)$$
$\mathcal{N}$ is a fixed hyperparameter that controls the learning rate/step size, set before training happens

1. Choose hyperparameter that looks reasonable, assumption: does not influence much
2. Choose hyperparameter from the literature, assumption: does matter, but best practices exist
3. Find best hyperparameter possible on own dataset

### Neural networks and Backpropagation
Network predicts output, computes error, backpropagation calculates how much each weight contributed to that error, then update weights to reduce error using gradient descent, applied layer by layer.


Multiclass classification, cannot use sigmoid output. We usually use a **softmax** function
$$softmax (z_{i}) = \frac{e^{z_{i}}} {\sum^C_{j=1} e^{z_{i}}}$$
Produces a probability distribution over all $C$ classes, the class with the highest probability is chosen as the prediction. Differentiable, so can still use backpropagation. 

### Deep learning
Same process, more layers and nonlinearities, each layer learn more abstract representations if input. Lower layers learn simple features like words, higher layers = complex patterns. Overfitting possible, expensive to train with lot of samples. Techniques to reduce overfitting include dropout(disable neurons randomly), early stopping (before convergence), and regularisation.

#### Regularisation
Process of finding best trade-off between simplicity and training error = regularisation

Want best compromise between training error and model complexity.

Modify the cost function to penalise complexity.
$$Cost(\beta) = Error(\beta, X, Y) + \lambda \cdot Complexity(\beta)$$
where:
	$\lambda$ : regularisation parameter controlling penalty strength
	$Complexity(\beta)$: depends on the size of the weights

Common ways of regularising linear models:$$
\text{Complexity}(\boldsymbol{\theta}) = \lambda \sum_{i=1}^{n} \theta_i^2$$
*L2* regularisation


$$\text{Complexity}(\boldsymbol{\theta}) = \lambda \sum_{i=1}^{n} |\theta_i|$$
*L1* regularisation

$$ 
\text{Complexity}(\boldsymbol{\theta}) = 
\lambda_1 \sum_{i=1}^{n} \theta_i^2 + 
\lambda_2 \sum_{i=1}^{n} |\theta_i|$$
Elastic net regularisation



