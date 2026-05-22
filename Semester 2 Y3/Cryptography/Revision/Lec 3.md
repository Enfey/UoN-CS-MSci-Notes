## Modern stream ciphers
- Aim to approximate the one-time pad's properties in aid of perfect secrecy, but without the drawbacks
- Used fixed length initial seed key to generate infinite pseudorandom keystream via CSPRNG, then use XOR.
- Practical
	- Init bys eed key, person on other side has same CSPRNG essentially, generate same keystream, due to properties of XOR it behaves as its own inverse, boom 


### Pseudorandomness
- There are three types of random number generators:
	- **True Random Number Generators(TRNGs)**
	- **Pseudorandom Number Generators**
	- **Cryptographically secure pseudorandom number generators**
- **TRNGs** impractical at scale
- **PRNGs** are fast and deterministic, but given enough output and literature knowledge one could uncover the rest of the keystream.
- **CSPRNGs** are deterministic but *computationally unpredictable* - given enough output a well-designed CSPRNG means an attacker could not recover further key bits, internal state, etc. 
	- Designed such that randomness is indisitnguishable from true random noise

### Flip-Flops
- Recall flip-flops; early stream ciphers were hardware oriented, clock-driven
- $D$ **Flip-flops** capture the value of $D-input$ at a definite portion of the clock cycle, typically the rising edge of the clock
	- Simply reflect the input into the output depending on the clock pulse.
		![](Pasted%20image%2020260210180339.png)
	- This captured output becomes the $Q$ output, and when the clock does not rise, the $Q$ output remains the same
- Hold a single bit of information
- But we can set up a circuit of these in the form of **linear feedback shift registers** to produuce statistically random looking data.


| Clock       | $D$ | $Q_{next}$ |
| ----------- | --- | ---------- |
| Rising edge | 0   | 0          |
| Rising edge | 1   | 1          |
| Non-rising  | X   | $Q$        |

## Linear feedback shift registers
- A **linear feedback shift register** is an $m$-**bit** register that shifts bits to the right every clock cycle, and computes a new input bit(the **feedback**) to the $LSFR$ as the $XOR$ of some bits of the overall shift register value. 
	$D-input$ comes from circuit and from feedback bit. 
- Usually comprised as flip flops, where the last bit stored for each clock cycle represents the output bit. 
	![](Pasted%20image%2020260210181558.png)
	Clocking this generates a sequence of output bits. 
- We can make this more complex, we are not limited by the number of flip-flops, or feedback complexity. 
	![](Pasted%20image%2020260210181929.png)
- These LSFRs are not good, we added another XOR gate to compute feedback, but imagine we're using a 128 bit LSFR to produce random looking data, but actually only have a sequence length of 10?
- Must design a good LSFR.


### Mathematical representation of LSFRs
- The most general form of an LSFR looks like so:
	![](Pasted%20image%2020260210185909.png)
- We have $m$ flip-flops for an $m-bit$ $LSFR$
	With state $s_{m-1}, s_{m-2} \dots, s_{1}, s_{0}$ at time $t$ according to $\{0, 1\}^m$ 
- The rightmost bit $s_0$ is the output bit
- Selected bits are fed back through the XOR gates to form the new input bit. 
- The symbols $p_0, p_1, \dots, p_{m-1}$ are called taps
	- Each $p_i \in \{0, 1\}$, if $p_i = 1$ then that register contributes to feedback, if $p_i = 0$ then that register does not contribute to feedback. The symbol is multiply. 
- The equations describe how each input bit is a linear combination of previous bits
	- Because they are linear equations that describe, with tap on/off coefficients, the registers that contribute to the feedback, this provides an attack vector.

### Polynomial reprtesentations of LSFRs
- We usually represent $m$ bit LSFRs using polynomials of degree $m$
	$$P(x) = x^m + p_{m-1}x^{m-1}+ \dots + p_{1}x+p_0$$ ![](Pasted%20image%2020260521175053.png)
Where $p_i$ act as coefficients denoting which taps are enabled, thus, which registers contribute to feedback and are not ignored. 



### Maximum length LSFRs
- When is an LSFR 'good'?
- An **m-bit** LSFR can have at most $2^m-1$ states given that its state is binary
	- The all zero state loops forever, of course, so this is excluded
- An LSFR that achieves this maximum number of states is called a **maximum-length LSFR**
	![](Pasted%20image%2020260210191820.png)
	The key theorem is that an LSFR produces a maximal length sequence iff its feedback polynomial is **primitive**. 
- The **period** of an LSFR is the number os steps before the internal state repeats
	- Thus, the period of an LSFR is directly tied to the number of cycles before a keystream begins repeating itself(which is obviously a problem)

### Attacking LSFRs
- Assuming a **known plaintext** attack, the attacker knows $2^m$ plaintext bits e.g., HTTP request header
- Attacker also has the corresponding ciphertext in this attack model
- Via $XOR$ can retrieve some keystream bits as $c = k \oplus m$ thus $k = c \oplus m$ as XOR is associative
	- This means we can do the following:
	1. Calculate key bits
		$s_i \equiv y_i + x_i \ (mod \ 2)$ , $i \in 2^m$ 
	2. Reconstruct the LSFR
		$s_m = s_{m-1}p_{m-1} + \dots s_1p_1 + s_0p_0$ 
		Becomes a system of linear equations where the unknowns are the $p_i$ coefficients, means that internal state can be recovered.
- Thus given just known plaintext ciphertext pairs, one is able to reconstruct the LSFR and thus the keystream 
- No single LSFR over a linear XOR in $GF(2)$ is cryptographically secure, no matter how long. 


### Trivium
- The solution to LSFRs linearity is to combine LSFRs where we have some nonlinear combination of their outputs as the true output. 
- We may use an $AND$ gate as this is a nonlinear function over GF(2)
- In trivium
	- 3 LSFRS
		- 93 bit
		- 84 bit
		- 111 bits
	- Chosen to have incompatible periods
		- That is, they share no large common divisors, aiming for roughly GCD = 1
		- The reason for this is that if the periods align, the period of the overall system would be lower, and there are less distinct internal states reachable before repetition, preventing statistical bias
	- Trivium uses an 80 bit key, and an 80 bit nonce as initialisation, and we cannot represent this as a system of linear equations, so it cannotn be broken that way.




### ChaCha20
- Stream cipher dependent on a $key, nonce, blocknum$ combination
- Takes a 256 bit key, 64 bit nonce, and 54 bit block, to yield a keystream that is XOR'ed with the 512 bit block plaintext
	- Nonce can be extended bit size; nonce+keystream reuse is catastrophic; would generate same keystream for same plaintext
- ChaCha arranges its internal state as a 4x4 grid of 32 bit words
	![](Pasted%20image%2020260521183927.png)
	Where the top 4 are 4 constants $c_0 ... c_3$ 
	The middle 4 32 bit words are the key,
	The bottom row holds the nonce and the block number
	This is the initial state for one block of the keystream
- ChaCha20 takes this grid and scrambles it, constructing from a tiny operation called a **quarter round** which simply performs addition, rotation, and XOR
	- All invertible, and fast, which kills side-channel timing attacks; e.g., may be fast if keystream is initially a power of two. 
	- Introduce non-linearity
	- Rotation introduces diffusion in combination with addition
	- and the operation as a whole introduces confusion
- ChaCha20 performs 20 rounds, in which XOR, rotate, and ADD take place, in 4 quarter rounds per round. 
	- Rounds alternate between **column rounds** and **diagonal rounds**, which affect where the quarter rounds apply to.
	- After a **double round** (column followed by diagonal) diffusion is achieved, as the words spread their influence to adjacent rows they otherwise would not have touched. 
- The final step is to add word-wise (2^32) with the input at the very end; this is because the operations carried out are reversible; iif you know the output of a quarter round for example, you can run it backward to acquire the original state. 
	- Now, the attacker would have to subtract and then reverse the operations; but the initial state contains the key they are trying to find, which has now been confused into the keystream and is no longer recoverable.
	- 




