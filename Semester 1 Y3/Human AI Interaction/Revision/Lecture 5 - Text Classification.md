<mark style="background: #FFF3A3A6;">Documents in corpus</mark>, <mark style="background: #FFF3A3A6;">represented, </mark><mark style="background: #FFF3A3A6;">TF-IDF weighting</mark> of their <mark style="background: #FFF3A3A6;">positive + negative words</mark>, <mark style="background: #FFF3A3A6;">yielding representation</mark> in <mark style="background: #FFF3A3A6;">2 dimensional vector space</mark> after <mark style="background: #FFF3A3A6;">preprocessing and representation</mark>. <mark style="background: #FFF3A3A6;">Sentiment classes</mark> are <mark style="background: #FFF3A3A6;">positive</mark>, and <mark style="background: #FFF3A3A6;">negative.</mark> 

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
<mark style="background: #FFF3A3A6;">How</mark> can we <mark style="background: #FFF3A3A6;">separate</mark> <mark style="background: #FFF3A3A6;">these two classes</mark> + <mark style="background: #FFF3A3A6;">classify documents</mark> pos/neg <mark style="background: #FFF3A3A6;">based on features?</mark>


## Classifying Text
- <mark style="background: #FFF3A3A6;">Goal</mark> = <mark style="background: #FFF3A3A6;">find rule</mark>/model, <mark style="background: #FFF3A3A6;">that</mark> can <mark style="background: #FFF3A3A6;">separate classes</mark>,<mark style="background: #FFF3A3A6;"> generalise</mark>

### Memorisation
<mark style="background: #FFF3A3A6;">Store exact samples</mark>, <mark style="background: #FFF3A3A6;">perfect</mark> on <mark style="background: #FFF3A3A6;">training data</mark>,<mark style="background: #FFF3A3A6;"> cannot generalise.</mark>

### OneR
<mark style="background: #FFF3A3A6;">Derives simple rule</mark> e.g., <mark style="background: #FFF3A3A6;">if x > 3</mark> then <mark style="background: #FFF3A3A6;">negative</mark>, otherwise positive. <mark style="background: #FFF3A3A6;">Easily interpretable,</mark> can <mark style="background: #FFF3A3A6;">generalise</mark> (<mark style="background: #FFF3A3A6;">poorly</mark>).

### Linear Classifier
<mark style="background: #FFF3A3A6;">Draws line</mark>, <mark style="background: #FFF3A3A6;">separate classes</mark>. Fast, <mark style="background: #FFF3A3A6;">generalises well, </mark><mark style="background: #FFF3A3A6;">limited to linear separability. </mark>
 $$y = \theta_{0} + \sum_{i=1}^n\theta_{i}x_{i}$$
 
This <mark style="background: #FFF3A3A6;">states</mark> that the <mark style="background: #FFF3A3A6;">output y</mark> is a <mark style="background: #FFF3A3A6;">weighted sum</mark> of the <mark style="background: #FFF3A3A6;">input features</mark> plus a bias term $\theta_{0}$. <mark style="background: #FFF3A3A6;">Yield line</mark>, <mark style="background: #FFF3A3A6;">plane,</mark> or <mark style="background: #FFF3A3A6;">hyperplane</mark>. <mark style="background: #FFF3A3A6;">Learned weights shared across all vectors.</mark>

### Nearest neighbour classifier
- <mark style="background: #FFF3A3A6;">Classify</mark>,<mark style="background: #FFF3A3A6;"> based</mark> on <mark style="background: #FFF3A3A6;">neighbourhood,</mark> <mark style="background: #FFF3A3A6;">new data,</mark> <mark style="background: #FFF3A3A6;">classified</mark>, <mark style="background: #FFF3A3A6;">based on closest neighbouring points</mark>. <mark style="background: #FFF3A3A6;">Low level of generalisation error</mark>, <mark style="background: #FFF3A3A6;">speed depends</mark> on the model.![[Pasted image 20251105100752.png]]
### Decision tree classifier
<mark style="background: #FFF3A3A6;">Split data</mark> by <mark style="background: #FFF3A3A6;">thresholds</mark>, can be <mark style="background: #FFF3A3A6;">efficient</mark>, <mark style="background: #FFF3A3A6;">speed depends</mark> a lot on <mark style="background: #FFF3A3A6;">model </mark>little space, <mark style="background: #FFF3A3A6;">generalise well</mark> but <mark style="background: #FFF3A3A6;">can overfit.</mark>
![[Pasted image 20251105100837.png]]

### Classification as generalisation
<mark style="background: #FFF3A3A6;">Core assumption</mark>, <mark style="background: #FFF3A3A6;">data</mark> is <mark style="background: #FFF3A3A6;">representative</mark> of <mark style="background: #FFF3A3A6;">real world</mark>. 
<mark style="background: #FFF3A3A6;">Generalisation challenge;</mark> <mark style="background: #FFF3A3A6;">vocabulary shifts</mark>, <mark style="background: #FFF3A3A6;">geographical</mark>, <mark style="background: #FFF3A3A6;">linguistic, </mark><mark style="background: #FFF3A3A6;">cultural differences,</mark> <mark style="background: #FFF3A3A6;">temporal shifts</mark> in the data. <mark style="background: #FFF3A3A6;">Never observe</mark> the r<mark style="background: #FFF3A3A6;">eal world</mark>, <mark style="background: #FFF3A3A6;">only sample;</mark> must ensure model assumptions, match reality.


## Machine learning
<mark style="background: #FFF3A3A6;">Learning via examples,</mark> <mark style="background: #FFF3A3A6;">discover patterns</mark> in data, <mark style="background: #FFF3A3A6;">generalise</mark> to <mark style="background: #FFF3A3A6;">new, unseen.</mark> <mark style="background: #FFF3A3A6;">Text classification</mark>, <mark style="background: #FFF3A3A6;">learn decision boundaries</mark> to <mark style="background: #FFF3A3A6;">separate categories. </mark>

### Training, testing, generalisation
<mark style="background: #FFF3A3A6;">Three types</mark> of <mark style="background: #FFF3A3A6;">error</mark> introduced
1. <mark style="background: #FFF3A3A6;">Training error </mark>- <mark style="background: #FFF3A3A6;">how well</mark> <mark style="background: #FFF3A3A6;">model fits</mark> <mark style="background: #FFF3A3A6;">training data</mark>
2. <mark style="background: #FFF3A3A6;">Testing error</mark> - <mark style="background: #FFF3A3A6;">how well</mark> <mark style="background: #FFF3A3A6;">model performs</mark> on <mark style="background: #FFF3A3A6;">unseen testing data</mark>
3. <mark style="background: #FFF3A3A6;">Generalisation error -</mark> <mark style="background: #FFF3A3A6;">how well</mark> <mark style="background: #FFF3A3A6;">model performs</mark> in<mark style="background: #FFF3A3A6;"> real world</mark>
<mark style="background: #FFF3A3A6;">Optimise generalisation error; </mark><mark style="background: #FFF3A3A6;">cannot measure</mark> it, so<mark style="background: #FFF3A3A6;"> use other 2</mark> and <mark style="background: #FFF3A3A6;">experimental design </mark>to <mark style="background: #FFF3A3A6;">estimate</mark> it. 
- **<mark style="background: #FFF3A3A6;">Train/test split</mark>** - reserve part of data for testing only
- **<mark style="background: #FFF3A3A6;">Cross-validation</mark>** - <mark style="background: #FFF3A3A6;">repeatedly train/test</mark> on <mark style="background: #FFF3A3A6;">diff partitions</mark>, get <mark style="background: #FFF3A3A6;">stable estimate</mark>
- **<mark style="background: #FFF3A3A6;">Bootstrapping</mark>**- <mark style="background: #FFF3A3A6;">sample data</mark> with <mark style="background: #FFF3A3A6;">replacement,</mark> create <mark style="background: #FFF3A3A6;">multiple train/test sets.</mark>

### Types of ML models for text classification
#### Traditional ML models
- Require <mark style="background: #FFF3A3A6;">manual feature extraction</mark> e.g., <mark style="background: #FFF3A3A6;">BOW, TF-IDF</mark>
	-<mark style="background: #FFF3A3A6;"> Linear models</mark> - <mark style="background: #FFF3A3A6;">LR</mark>
	- <mark style="background: #FFF3A3A6;">Tree models</mark> - decision trees
	- <mark style="background: #FFF3A3A6;">Probabilistic models</mark> - <mark style="background: #FFF3A3A6;">naive bayes</mark>
	- KNN
-<mark style="background: #FFF3A3A6;"> Representation </mark>is<mark style="background: #FFF3A3A6;"> user defined</mark>, fed to model.
#### Deep learning models
<mark style="background: #FFF3A3A6;">Learn representations automatically</mark> via <mark style="background: #FFF3A3A6;">multiple layers</mark>; <mark style="background: #FFF3A3A6;">perceptron</mark>/<mark style="background: #FFF3A3A6;">multi-layer perceptron</mark>, <mark style="background: #FFF3A3A6;">CNNs</mark>, <mark style="background: #FFF3A3A6;">RNNs</mark>, <mark style="background: #FFF3A3A6;">Transformers</mark>.

### ML approaches
- **<mark style="background: #FFF3A3A6;">Supervised</mark>** - <mark style="background: #FFF3A3A6;">known ground truth</mark> about training data
- **<mark style="background: #FFF3A3A6;">Unsupervised</mark>** - <mark style="background: #FFF3A3A6;">no ground truth</mark> about the data
- **<mark style="background: #FFF3A3A6;">Semi-supervised</mark>** - ground truth about some of data
- **<mark style="background: #FFF3A3A6;">Reinforcement learning</mark>** - technique, <mark style="background: #FFF3A3A6;">agent learns</mark>, <mark style="background: #FFF3A3A6;">make decisions</mark> via<mark style="background: #FFF3A3A6;"> trial+error</mark> to <mark style="background: #FFF3A3A6;">maximise reward signal</mark>, receives feedback in form of <mark style="background: #FFF3A3A6;">rewards/penalties</mark>. 

## Learning and optimisation
At <mark style="background: #FFF3A3A6;">heart of ML</mark>, <mark style="background: #FFF3A3A6;">learning</mark>, <mark style="background: #FFF3A3A6;">model parameters</mark><mark style="background: #FFF3A3A6;"> tuned </mark>to <mark style="background: #FFF3A3A6;">minimise loss function.
</mark>
<mark style="background: #FFF3A3A6;">Perceptron</mark>, <mark style="background: #FFF3A3A6;">preceded modern NN.</mark>
$$y = f(\sum_{i}w_{i}x_{i}+b)$$
Where $f$ is non-linear activation.
<mark style="background: #FFF3A3A6;">Typical functions:</mark>
<mark style="background: #FFF3A3A6;">Step function</mark>: $f(x) = 1 \ if \ \frac{1}{1+e^{-x}} \geq 0 \ else -1$ 
<mark style="background: #FFF3A3A6;">Sigmoid</mark>: $f(x) = \frac{1}{1+e^{-x}}$
<mark style="background: #FFF3A3A6;">Hyperbolic Tangent</mark>: $f(x) = tanh(w \cdot x + b)$ 
<mark style="background: #FFF3A3A6;">ReLU</mark>: $f(x) = max(0, x)$ 

Given inputs $x_1\dots x_{n}$ and weights $w_1\dots w_{n}$ <mark style="background: #FFF3A3A6;">model outputs</mark> $y$ and <mark style="background: #FFF3A3A6;">compares to true label </mark>to <mark style="background: #FFF3A3A6;">compute loss.</mark> 

New input $p_1$ represented by 3 features.

$p_1: x_1 = 0.3, x_2 = 0.3, x_3 = 0.3$
$p_1 : w_1 = 1.5, w_2 = -1.5, w_3 = 3.3, w_b = 0.3$ 

f(1.29)
$\frac{1}{1+e^{1.29}} = 0.78471$ using step function.
Decision boundary reached, <mark style="background: #FFF3A3A6;">classified as 1. </mark>

### Learning via gradient descent
<mark style="background: #FFF3A3A6;">Once model</mark> starts <mark style="background: #FFF3A3A6;">making predictions</mark>, needs to <mark style="background: #FFF3A3A6;">learn better weights</mark>, where <mark style="background: #FFF3A3A6;">loss function</mark> + <mark style="background: #FFF3A3A6;">gradient descent</mark> come in. 

<mark style="background: #FFF3A3A6;">Loss function </mark>$\mathcal{J}$<mark style="background: #FFF3A3A6;"> measures</mark> h<mark style="background: #FFF3A3A6;">ow incorrect model predictions are</mark>, minimise.

<mark style="background: #FFF3A3A6;">Iteratively</mark> <mark style="background: #FFF3A3A6;">update parameters</mark> $w$ <mark style="background: #FFF3A3A6;">over</mark> $n$ <mark style="background: #FFF3A3A6;">iterations</mark> using the <mark style="background: #FFF3A3A6;">partial derivate</mark> of<mark style="background: #FFF3A3A6;"> loss function</mark> with<mark style="background: #FFF3A3A6;"> respect to the parameters.</mark>


$$w_{t+1} = w^t - \mathcal{N} \cdot \Delta_{w^t} \mathcal{J}(w^t)$$
$\mathcal{N}$ is a fixed hyperparameter that<mark style="background: #FFF3A3A6;"> controls the learning rate</mark>/step size, <mark style="background: #FFF3A3A6;">set before training happens</mark>

1. <mark style="background: #FFF3A3A6;">Choose hyperparameter</mark> that <mark style="background: #FFF3A3A6;">looks reasonable</mark>, <mark style="background: #FFF3A3A6;">assumption</mark>: <mark style="background: #FFF3A3A6;">does not influence much</mark>
2. <mark style="background: #FFF3A3A6;">Choose hyperparameter</mark> from the<mark style="background: #FFF3A3A6;"> literature</mark>, assumption: d<mark style="background: #FFF3A3A6;">oes matter</mark>, but <mark style="background: #FFF3A3A6;">best practices exist</mark>
3. <mark style="background: #FFF3A3A6;">Find best hyperparameter possible</mark> on<mark style="background: #FFF3A3A6;"> own dataset</mark>

### Neural networks and Backpropagation
<mark style="background: #FFF3A3A6;">Network predicts output,</mark> c<mark style="background: #FFF3A3A6;">omputes error,</mark> <mark style="background: #FFF3A3A6;">backpropagation </mark><mark style="background: #FFF3A3A6;">calculates</mark> <mark style="background: #FFF3A3A6;">how much each weight </mark><mark style="background: #FFF3A3A6;">contributed</mark> <mark style="background: #FFF3A3A6;">to</mark> that <mark style="background: #FFF3A3A6;">error,</mark> then<mark style="background: #FFF3A3A6;"> update weights</mark> to <mark style="background: #FFF3A3A6;">reduce error</mark> using <mark style="background: #FFF3A3A6;">gradient descent</mark>, applied <mark style="background: #FFF3A3A6;">layer by layer.</mark>


<mark style="background: #FFF3A3A6;">Multiclass classification</mark>, <mark style="background: #FFF3A3A6;">cannot use sigmoid output.</mark> We usually use a **softmax** function
$$softmax (z_{i}) = \frac{e^{z_{i}}} {\sum^C_{j=1} e^{z_{i}}}$$
<mark style="background: #FFF3A3A6;">Produces a probability distribution</mark> over <mark style="background: #FFF3A3A6;">all $C$ classes</mark>, the <mark style="background: #FFF3A3A6;">class</mark> with the <mark style="background: #FFF3A3A6;">highest probability</mark> is <mark style="background: #FFF3A3A6;">chosen</mark> as the <mark style="background: #FFF3A3A6;">prediction</mark>. <mark style="background: #FFF3A3A6;">Differentiable</mark>, so can still use backpropagation. 

### Deep learning
<mark style="background: #FFF3A3A6;">Same process,</mark> <mark style="background: #FFF3A3A6;">more layers</mark> and <mark style="background: #FFF3A3A6;">nonlinearities,</mark> each<mark style="background: #FFF3A3A6;"> layer learn</mark> more <mark style="background: #FFF3A3A6;">abstract representations</mark> if input. <mark style="background: #FFF3A3A6;">Lower layers</mark><mark style="background: #FFF3A3A6;"> learn simple features</mark> like words, <mark style="background: #FFF3A3A6;">higher layers </mark>= <mark style="background: #FFF3A3A6;">complex patterns</mark>. <mark style="background: #FFF3A3A6;">Overfitting </mark><mark style="background: #FFF3A3A6;">possible,</mark> <mark style="background: #FFF3A3A6;">expensive to train</mark> with lot of samples.

<mark style="background: #FFF3A3A6;">Techniques</mark> to <mark style="background: #FFF3A3A6;">reduce overfitting</mark> <mark style="background: #FFF3A3A6;">include dropout</mark>(disable neurons randomly), <mark style="background: #FFF3A3A6;">early stopping</mark> (before convergence), <mark style="background: #FFF3A3A6;">and regularisation.</mark>

#### Regularisation
<mark style="background: #FFF3A3A6;">Process </mark>of <mark style="background: #FFF3A3A6;">finding</mark> best <mark style="background: #FFF3A3A6;">trade-off</mark> between <mark style="background: #FFF3A3A6;">simplicity</mark> and <mark style="background: #FFF3A3A6;">training error</mark> = regularisation

<mark style="background: #FFF3A3A6;">Want best compromise</mark> <mark style="background: #FFF3A3A6;">between training error</mark> and <mark style="background: #FFF3A3A6;">model complexity.</mark>

<mark style="background: #FFF3A3A6;">Modify the cost function</mark> to <mark style="background: #FFF3A3A6;">penalise complexity.</mark>
$$Cost(\beta) = Error(\beta, X, Y) + \lambda \cdot Complexity(\beta)$$
where:
	$\lambda$ : <mark style="background: #FFF3A3A6;">regularisation parameter</mark> <mark style="background: #FFF3A3A6;">controlling</mark> <mark style="background: #FFF3A3A6;">penalty strength</mark>
	$Complexity(\beta)$: <mark style="background: #FFF3A3A6;">depends</mark> on the <mark style="background: #FFF3A3A6;">size of </mark>the <mark style="background: #FFF3A3A6;">weights</mark>

<mark style="background: #FFF3A3A6;">Common ways</mark> of regularising linear models:$$
\text{Complexity}(\boldsymbol{\theta}) = \lambda \sum_{i=1}^{n} \theta_i^2$$
<mark style="background: #FFF3A3A6;">*L2* regularisation</mark>


$$\text{Complexity}(\boldsymbol{\theta}) = \lambda \sum_{i=1}^{n} |\theta_i|$$
<mark style="background: #FFF3A3A6;">*L1* regularisation</mark>

$$ 
\text{Complexity}(\boldsymbol{\theta}) = 
\lambda_1 \sum_{i=1}^{n} \theta_i^2 + 
\lambda_2 \sum_{i=1}^{n} |\theta_i|$$
<mark style="background: #FFF3A3A6;">Elastic net regularisation</mark>



