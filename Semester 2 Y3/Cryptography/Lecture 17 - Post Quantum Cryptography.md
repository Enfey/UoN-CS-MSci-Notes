## Qubits
- A quantum bit is a physical system that represents a linear combination of basis states/vectors
- We typically denote this in **ket notation**, used to represent a a physical system's state as a column vector: $$\vert \psi \rangle = \alpha \vert 0 \rangle + \beta \vert 1 \rangle$$
	If we switch to vector form: $$  
|0\rangle =  
\begin{bmatrix}  
1 \\  
0  
\end{bmatrix},  
\quad  
|1\rangle =  
\begin{bmatrix}  
0 \\  
1  
\end{bmatrix}  
$$
	Then: $$|\psi\rangle =  
\begin{bmatrix}  
\alpha \\  
\beta  
\end{bmatrix}$$
- The coefficients $\alpha$ and $\beta$ are complex numbers (quantum systems evolve like rotations, and rotations are naturally expressed using complex numbers)
- The coefficients are not arbitrary, they must satisfy the following (they are probability amplitudes, and the probabilities must sum to one): $$\vert \alpha \vert ^2 + \vert \beta \vert ^2 = 1$$
- Geometrically then, a valid qubit is any unit vector in $\mathbb{C}^2$ 
- Qubits can be built in many different ways e.g., supercooled circuits, but we can abstract away from this entirely. 


## Quantum Superposition
- A fundamental principle in quantum computing that permits qubits to exist in multiple states simultaneously.
- A single cubit as we have observed, contains two states where $\alpha$ and $\beta$ are probability amplitudes influencing measurement probabilities
	- Two cubits then hold 4 states
	- $n$ qubits hold $2^n$ simultaneous possibilities.
- When a measurement is performed on a qubit in superposition, it "collapses" into one of its basis states, $\vert 0 \rangle$ or $\vert 1 \rangle$ 
	- The outcome of this collapse is probabilistic and is determined by the probability amplitudes $\alpha$ and $\beta$. 
		![](Pasted%20image%2020260331232158.png)
- When you measure a system of multiple qubits, you get one of $2^n$ outcomes
	- Say for two cubits:
		![](Pasted%20image%2020260331232430.png)
	- The coefficients sum to probability 1 (each probability is $|1/2|² = 1/4$) , and when measuring, we get the entire 2 bit result at once.
	- Therefore quantum algorithms are designed to make the right answer the most probable outcome before measurement. 

## Quantum Operations
- Quantum operations are a linear transformation applied to a qubit, that have the potential to alter probability amplitudes
- **Quantum gates** are the quantum equivalent of a classical logic gate. Each gate corresponds to a small unitary matrix $U$ such that: $$\vert \psi ' \rangle = U \vert \psi \rangle$$
	- Acts on one or more qubits. 
	- Examples include:
		- **Identity** 
			- Does absolutely nothing, exists for timing/alignment purposes; NOP.
		- **Hadamard gate**
			- Creates superposition; it turns a definite state $\vert 0 \rangle$ into $H \vert \rangle = \frac{1}{\sqrt 2}(\vert 0 \rangle + \vert 1 \rangle)$  
				![](Pasted%20image%2020260331233633.png)
		- **NOT**
			- The $X$ gate is:
				![](Pasted%20image%2020260331233648.png)
			- Simply inverts a qubit
				![](Pasted%20image%2020260331233709.png)
		- **CNOT**
	- Because the matrices are unitary, all operations can be inverted.
- **Quantum circuits** combine gates into an algorithm. Since each gate is a matrix, and we apply them in sequence, we just multiply all the matrices together to yield a matrix $U_{c}$ for that particular circuit.
	- Note that for a single qubit, gates are $2 \times 2$ matrices
	- When we have $n$ qubits, the full system lives in $2^n$ dimensional space, so the total matrix is $2^n \times 2^n$ 

### What quantum is not
![](Pasted%20image%2020260331234153.png)
- Here we have a circuit with 256 qubits initialised to $\vert 0 \rangle$ 
- We use $H$ gates to create a superposition of all possible $2^n$ inputs
	- The individual gates here are $2 \times 2$ but the system they collectively operate on is $2^n$ dimensional.
- We produce a uniform circuit $U_f$ that computes SHA-256 on this superposition
 - We do not do this. We must design algorithms that amplify the likelihood of getting the correct answer when we measure and reduce the chance of the correct answer.

### Quantum Algorithms
- **Grover's algorithm**
	- Grover's algorithm is used for unstructured search i.e., brute force
		- Guessing the secret key, guessing the pre-image of a hash etc.
	- For example, finding the value of $k$ such that $E^{-1}_{k}(y) =x$ 
	- We must try keys one at a time classically, which takes $O(n)$
	- Grovers works like so:
		- We use 128 qubits which can represent $2^{128}$ basis states
		- We prepare the qubits in equal superposition. 
			- All keys exist simultaneously in the quantum state
		- We send the qubits to an oracle, which based on a function, flips the phase(sign) of the amplitude of the correct key's basis state. Each grover iteration increases the amplitude of the correct state, making final measurement likely to get the correct answer.
			![](Pasted%20image%2020260401000033.png)
		- Requires $\sqrt {2^n}$ iterations
		- 128 bits would be breakable in $2^{64}$ which is feasible, but 256 bit algorithms are not yet breakable.
		- Grover's has no effect on asymmetric algorithms like RSA.
- **Shor's Algorithm**
	- Exponential speedup on all asymmetric algorithms
	- ECDHE, DHE, DSA+RSA all broken
	- No effect on symmetric algorithms or hash functions


### Current State of Quantum Hardware
- Current state of the art is roughly < 50 logical qubits (1 logical qubit can require 100 to 1000 physical qubits to overcome noise that simulate one robust qubit, behaving as an idealised qubit)
- Current hardware however has extremely high error rates (0.1-1% per gate). Running Shor's to break RSA 2048 would 4000 logical qubits, and millions of physical qubits.
- **The optimist view:**
	- Progress has been rapid
	- We know of no physical law that prevents us from scaling up the amount of physical qubits in a quantum computer
	- We're exploring multiple physical modalities to instantiate physical qubits, so that if one of them doesn't work, one of them will and allow us to scale.
- **The pessimist view**
	- Overcoming errors at scale is perhaps deemed to be impossible, meaning that algorithms will not execute as intended.
	- Qubits decohere too rapidly; when we put a qubit into a quntum superposition, it will last in that superposition in the order of nano or milliseconds. 
		- If the algorithm takes more than a few milliseconds, we don't know what to do. 

### Threat Model - Harvest Now, Decrypt Later
- An adversary (nation state) captures encrypted traffic today - TLS handshakes , VPN sessions and traffic, TLS session data
- Stores it cheaply - inexpensive at scale
- Wait until a capable quantum computer exists
- Decrypt chosen traffic retroactively
	- E.g., Shor's breaks ECDHE, TLS, get the pre-master secret and can then get the session key and decrypt everything as all the information needed would've been in the recorded handshake.
- PRISM - the NSA bulk collection program revealed by Snowden in 2013 confirmed that nation states already have both the infrastructure and authority to intercept and store massive internet traffic at scale. 
- Post-quantum cryptographic standards now exist; the migration is achieveable and low cost vs high reward.

### Post-Quantum Algorithms
- There are 3 broad categories of scheme that we think are resistant to quantum attacks, as in, there are no known quantum algorithms that can solve these problems:
1. **Lattice-based problems** (hard geometric problems in high-dimensions)
2. **Hash-based schemes**
3. **Code-based schemes**
![](Pasted%20image%2020260401005638.png)



## Lattice-Based Cryptography
- A **lattice** is a linear integer combination of basis vectors: $$ L = \{x_{1}a_{1} + x_{2}a_{2} + \dots + x_{n}a_{n}\}$$
	Where $x \in \mathbb{Z}$; thus a lattice is every point reachable by all possible linear integer combinations of basis vectors.
- The rank $n$ of the lattice is the number of linearly independent basis vectors (the scaled basis-vectors) generating the lattice. 
- The vectors $a_i$ are of dimension $m$
- The rank and dimension of a lattice are not always equal; a lattice is defined by its rank $n$, and they are equal only in a **full-rank lattice** where the number of basis vectors equals the dimension of the space.
	- E.g., a $2D$ lattice in $3D$ space has rank $2$ and dimension $3$.
- Oftentimes the integer coeffieicents are calculated mod $q$ to 'box' the lattice and make it wrap around. 
	- This is what we call a $q-ary$ lattice which exists on a torus.

### Lattice Geometry
- A set of basis vectors defines a periodic (repeats forever (or until $q$)) set of points one can reach in the lattice according to the corresponding integer coefficients $x_i$ 
- Points are reached through different linear combinations of these basis vectors:
	![](Pasted%20image%2020260401020315.png)
- Equivalent lattices *can* be produced by different bases (that is, the two bases can reach every single one of the same points under $q$, and some bases can only reach some of the same lattice points/co-ordinates in $m$ dimensional space, and thus form a **sub-lattice**) but some bases are 'easier' to use than others, take for example the below which is able to describe the same point, but requires greater integer coefficients:
	![](Pasted%20image%2020260401020611.png)
### Lattice problems
- **Closest Vector Problem** (CVP)
	- Foundational NP-hard optimisation problem
	- Involves finding the lattice point $p$ within a lattice $\mathcal{L}$ that is closest to some target vector $t$ (which may not necessarily be $\in \mathcal{L}$, but may reside in the outer vector space $R^m$ )
	- This is NP-hard with a bad basis.
		- A bad basis consists of long, nearly parallel vectors.
		- The hardness comes from the fact that not only would one need to acquire a basis that can reach the closest point on the lattice to point in vector space, but would then also need to actually compute said point.
			- The attacker cannot efficiently navigate the lattice; and even if they knew which lattice point was closest e.g., the integer coefficients, they would still need to actually compute the co-ordinates in real space.
- **Shortest Vector problem**
	- Given a lattice $\mathcal{L}$ with basis $B = \{b_1, b_2, \dots, b_n\}$ spanning an ambient space $R^m$ find the shortest non-zero point on the lattice $p$ such that $p = \sum^n_i x_i b_i, \ x_i \in Z, \ \ p \neq 0$ 
	- The structural difference is that the target is the origin, which is $\in \mathcal{L}$ but the solution is constrained to be a non-trivial lattice point. 
	- This is also NP-hard with a bad basis, for the exact same reasons as above.
		- One would need to acquire a basis that can actually reach the closest lattice point to the origin (may not even generate a sub-lattice), but would also need to then compute said point. 
			- Cannot efficiently navigate the lattice without correct bases.
			- For thousands of dimensions and decently sized $q$, computing this lattice point with no information on bases is nigh-on impossible.
- Note that in practice, lattices usually contain thousands of dimensions, and we do not know all the lattice points without computing them, that is, we cannot simply enumerate the lattice points and use a distance metric $M$ to solve our problem.
- The CVP is a generalistion of the SVP, and given an oracle for the CVP one can solve the SVP by making some queries to the oracle.
- We usually choose bases randomly for high likelihood of difficulty.
	- The LLL algorithm is a basis reduction algorithm that attempts to find shorter, more orthogonal basis vectors that span the same lattice.
	- In low dimensions, LLL gets close enough to be dangerous, which motivates the high-dimensionality of lattice-based cryptography.

### Kyber, Falcon, and Dilithium
- Utilise a shared primitive:
	- **Learning with errors** - represents a secret as an equation with errors.
		- Let $\mathbb{Z}_q$ denote the ring of integers modulo $q$ and let $\mathbb{Z}^m_q$ denote the set of $m$-vectors over $\mathbb{Z}_q$. 
		- Pick a secret vector $s \in \mathbb{Z}^m_q$, that can be thought of as the integer coefficients in the linear combination of A's columns
		- Generate a random matrix $A \in \mathbb{Z}^{m \times n}_q$ representing the basis column vectors of the lattice and compute: $$b = As + e \ (mod \ q)$$
			Where:
			- $e$ is a small error vector
			- $As$ is a point on the lattice
		- $b \in Z^m_q$ is a vector that is close to, but not on the lattice $\mathcal{L}(A$), as the $e$ term perturbs $As$ off the lattice.
		- $e \in \mathbb{Z}^m$ is sampled from a discrete Gaussian Distribution ensuring that $e$ is small and does not push $b$ too far from any lattice point.
- The public key then is $A, b$. The hardness assumption is that given $A, b$, recovering the secret $s$ is computationally infeasible. The error vector pushes $b$ off the lattice defined by $A$, instantiating CVP hardness as the lattice would need to find the closest lattice point to $b$ under bad basis. 
![](Pasted%20image%2020260401060729.png)


## Hash-Based signatures
- Hash functions like SHA-256 and SHA-3 have been subjected to decades of cryptanalysis by the entire cryptographic community
- This is in stark contrast to lattice-based assumptions
- Building signatures purely from hash functions therefore gives a reliable security foundation
- All schemes are built around this premise:
	"*Prove you authored a message by revealing hashed pre-images that you committed to earlier*"
- This works because of two hash function properties
	- **Pre-image resistance**: given $y = H(x)$, you cannot invert and find $x$
	- **Second pre-image resistance**: given $x$ and $H(x)$, cannot find $x' \neq x$ such that $H(x') = H(x)$ 
- Publishing $y = H(x)$ is a commitment, as you have bound yourself to $x$ without revealing it.
	- Later revealing $x$ proves you knew it before publishing $y$.
		- Assuming the hash properties above, that is.


### One-Time Signatures
- 

### Winternitz OTS

### Merkle Trees

### SPHINCS+

## Hybrid Schemes
