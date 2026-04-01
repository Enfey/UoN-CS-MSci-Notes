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
			![](Pasted%20image%2020260331235937.png)
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
- 

### Kyber, Falcon, and Dilithium


## Hash-Based signatures

