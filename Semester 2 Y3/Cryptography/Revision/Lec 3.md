## Modern stream ciphers
- <mark style="background: #FFF3A3A6;">Aim to approximate</mark> the <mark style="background: #FFF3A3A6;">one-time pad'</mark>s properties in aid of perfect secrecy, but without the drawbacks
- <mark style="background: #FFF3A3A6;">Used fixed length initial seed key to generate infinite pseudorandom keystream</mark> via CSPRNG, then <mark style="background: #FFF3A3A6;">use XOR</mark>.
- Practical
	- Init bys eed key, person on other side has same CSPRNG essentially, generate same keystream, due to properties of XOR it behaves as its own inverse, boom 


### Pseudorandomness
- There are three types of random number generators:
	- **True Random Number Generators(TRNGs)**
	- **Pseudorandom Number Generators**
	- **Cryptographically secure pseudorandom number generators**
- **TRNGs** impractical at scale
- **PRNGs** are fast and deterministic, but given enough output and literature knowledge one could uncover the rest of the keystream.
- **CSPRNGs** are deterministic but *c<mark style="background: #FFF3A3A6;">omputationally unpredictable*</mark> - given enough output a well-designed CSPRNG means an attacker could not recover further key bits, internal state, etc. 
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
- A **linear feedback shift register** is an $m$-**bit** <mark style="background: #FFF3A3A6;">register</mark> that<mark style="background: #FFF3A3A6;"> shifts bits to the right </mark>every clock cycle, and c<mark style="background: #FFF3A3A6;">omputes a new input bit</mark>(the **feedback**) to the $LSFR$ as the $XOR$ o<mark style="background: #FFF3A3A6;">f some bits of the overall shift register value. </mark>
	$D-input$ comes from circuit and from feedback bit. 
- Usually comprised as flip flops, where the <mark style="background: #FFF3A3A6;">last bit stored for each clock cycle represents the output bit</mark>. 
	![](Pasted%20image%2020260210181558.png)
	Clocking this generates a sequence of output bits. 
- We can make this more complex, we are <mark style="background: #FFF3A3A6;">not limited by the number of flip-flops, or feedback complexity. </mark>
	![](Pasted%20image%2020260210181929.png)
- <mark style="background: #FFF3A3A6;">These LSFRs are not good</mark>, we added another XOR gate to compute feedback, but imagine we're using a 128 bit LSFR to produce random looking data, but actually only have a sequence length of 10?
- <mark style="background: #FFF3A3A6;">Must design a good LSFR.</mark>


### Mathematical representation of LSFRs
- The <mark style="background: #FFF3A3A6;">most general form of an LSFR</mark> looks like so:
	![](Pasted%20image%2020260210185909.png)
- We have $m$ flip-flops for an $m-bit$ $LSFR$
	With state $s_{m-1}, s_{m-2} \dots, s_{1}, s_{0}$ at time $t$ according to $\{0, 1\}^m$ 
- The rightmost bit $s_0$ is the output bit
- <mark style="background: #FFF3A3A6;">Selected bits are fed back through the XOR gates to form the new input bit. </mark>
- The symbols $p_0, p_1, \dots, p_{m-1}$ are called taps
	- Each $p_i \in \{0, 1\}$, if $p_i = 1$ <mark style="background: #FFF3A3A6;">then that register contributes to feedback</mark>, if $p_i = 0$ then that <mark style="background: #FFF3A3A6;">register does not contribute to feedback</mark>. The symbol is multiply. 
- T<mark style="background: #FFF3A3A6;">he equations describe how each input bit is a linear combination of previous bits</mark>
	- <mark style="background: #FFF3A3A6;">Because they are linear equations that describe</mark>, with tap on/off coefficients, t<mark style="background: #FFF3A3A6;">he registers that contribute to the feedback</mark>, this<mark style="background: #FFF3A3A6;"> provides an attack vector.</mark>

### Polynomial reprtesentations of LSFRs
- We usually represent $m$ bit LSFRs using polynomials of degree $m$
	$$P(x) = x^m + p_{m-1}x^{m-1}+ \dots + p_{1}x+p_0$$ ![](Pasted%20image%2020260521175053.png)
Where $p_i$ <mark style="background: #FFF3A3A6;">act as coefficients denoting which taps are enabled,</mark> thus, which registers contribute to feedback and are not ignored. 



### Maximum length LSFRs
- When is an LSFR 'good'?
- An **m-bit** LSFR can have at most $2^m-1$ states <mark style="background: #FFF3A3A6;">given that its state is binary</mark>
	- The <mark style="background: #FFF3A3A6;">all zero state loops forever,</mark> of course, <mark style="background: #FFF3A3A6;">so this is excluded</mark>
- <mark style="background: #FFF3A3A6;">An LSFR that achieves this maximum number of states is called a </mark>**maximum-length LSFR**
	![](Pasted%20image%2020260210191820.png)
	<mark style="background: #FFF3A3A6;">The key theorem is that an LSFR produces a maximal length sequence iff its feedback polynomial is **primitive**</mark>. 
- The **period** of an LSFR is the <mark style="background: #FFF3A3A6;">number os steps before the internal state repeats</mark>
	- Thus, the <mark style="background: #FFF3A3A6;">period of an LSFR</mark> is<mark style="background: #FFF3A3A6;"> directly tied </mark>to the <mark style="background: #FFF3A3A6;">number of cycles </mark>before a<mark style="background: #FFF3A3A6;"> keystream begins repeating itself(</mark>which is obviously a problem)

### Attacking LSFRs
- <mark style="background: #FFF3A3A6;">Assuming</mark> a **known plaintext** attack, the attacker knows $2^m$ plaintext bits e.g., HTTP request header
- <mark style="background: #FFF3A3A6;">Attacker also has the corresponding ciphertext</mark> in this attack model
- Via $XOR$ <mark style="background: #FFF3A3A6;">can retrieve some keystream bits</mark> as $c = k \oplus m$ thus $k = c \oplus m$ as XOR is associative
	- <mark style="background: #FFF3A3A6;">This means we can do the following:</mark>
	1. Calculate key bits
		$s_i \equiv y_i + x_i \ (mod \ 2)$ , $i \in 2^m$ 
	2. Reconstruct the LSFR
		$s_m = s_{m-1}p_{m-1} + \dots s_1p_1 + s_0p_0$ 
		<mark style="background: #FFF3A3A6;">Becomes a system of linear equations</mark> where the <mark style="background: #FFF3A3A6;">unknowns are the </mark>$p_i$ <mark style="background: #FFF3A3A6;">coefficients</mark>, <mark style="background: #FFF3A3A6;">means that internal state</mark> can be <mark style="background: #FFF3A3A6;">recovered.</mark>
- Thus given just known plaintext ciphertext pairs, one is able to reconstruct the LSFR and thus the keystream 
-<mark style="background: #FFF3A3A6;"> No single LSFR over a linear XOR</mark> in $GF(2)$ is<mark style="background: #FFF3A3A6;"> cryptographically secure</mark>, no matter how long. 


### Trivium
- The solution to LSFRs linearity is to combine LSFRs where we have some nonlinear combination of their outputs as the true output. 
- <mark style="background: #FFF3A3A6;">We may use an</mark> $AND$ gate as this is a<mark style="background: #FFF3A3A6;"> nonlinear function</mark> over GF(2)
- In trivium
	- 3 LSFRS
		- 93 bit
		- 84 bit
		- 111 bits
	- <mark style="background: #FFF3A3A6;">Chosen to have incompatible periods</mark>
		- That is, they share no large common divisors, aiming for roughly GCD = 1
		- <mark style="background: #FFF3A3A6;">The reason for this is that if the periods align</mark>, the <mark style="background: #FFF3A3A6;">period of the overall system would be lower,</mark> and there are less distinct internal states reachable before repetition, preventing statistical bias
	- Trivium uses an 80 bit key, and an <mark style="background: #FFF3A3A6;">80 bit nonce as initialisation</mark>, and we <mark style="background: #FFF3A3A6;">cannot represent this as a system of linear equations, </mark>so it cannotn be broken that way.




### ChaCha20
- <mark style="background: #FFF3A3A6;">Stream cipher dependent</mark> on a $key, nonce, blocknum$ combination
- Takes a 256 bit key, 64 bit nonce, and 54 bit block, to yield a keystream that is XOR'ed with the 512 bit block plaintext
	- <mark style="background: #FFF3A3A6;">Nonce can be extended bit size</mark>; <mark style="background: #FFF3A3A6;">nonce+keystream reuse is catastrophic</mark>; would generate same keystream for same plaintext
- <mark style="background: #FFF3A3A6;">ChaCha</mark> <mark style="background: #FFF3A3A6;">arranges its internal state as a 4x4 grid of 32 bit words</mark>
	![](Pasted%20image%2020260521183927.png)
	Where the top 4 are 4 constants $c_0 ... c_3$ 
	The middle 4 32 bit words are the key,
	<mark style="background: #FFF3A3A6;">The bottom row holds the nonce and the block number</mark>
	<mark style="background: #FFF3A3A6;">This is the initial state for one block of the keystream</mark>
- ChaCha20 <mark style="background: #FFF3A3A6;">takes this grid and scrambles it</mark>, constructing from a tiny operation called a **quarter round** which simply performs addition, rotation, and XOR
	- <mark style="background: #FFF3A3A6;">All invertible, and fast, which kills side-channel timing</mark> attacks; e.g., <mark style="background: #FFF3A3A6;">may be fast if keystream is initially a power of two. </mark>
	- <mark style="background: #FFF3A3A6;">Introduce non-linearity</mark>
	- <mark style="background: #FFF3A3A6;">Rotation introduces diffusion in combination with addition</mark>
	- and the operation as a whole introduces confusion
- ChaCha20 performs <mark style="background: #FFF3A3A6;">20 rounds, </mark>in which <mark style="background: #FFF3A3A6;">XOR, rotate, and ADD take place</mark>, in 4 quarter rounds per round. 
	- <mark style="background: #FFF3A3A6;">Rounds alternate between</mark> **column rounds** and **diagonal rounds**, which affect where the quarter rounds apply to.
	- After a **double round** (column followed by diagonal) <mark style="background: #FFF3A3A6;">diffusion is achieved</mark>, as the <mark style="background: #FFF3A3A6;">words spread their influence to adjacent </mark>rows they otherwise would not have touched. 
- The final step is to <mark style="background: #FFF3A3A6;">add</mark> <mark style="background: #FFF3A3A6;">word-wise</mark> (2^32) with the i<mark style="background: #FFF3A3A6;">nput at the very end;</mark> this is because the <mark style="background: #FFF3A3A6;">operations carried out are reversible</mark>; iif you know the output of a quarter round for example, you <mark style="background: #FFF3A3A6;">can run it backward</mark> to <mark style="background: #FFF3A3A6;">acquire the original state</mark>. 
	- Now, the <mark style="background: #FFF3A3A6;">attacker would have to subtract</mark> and then <mark style="background: #FFF3A3A6;">reverse the operations</mark>; b<mark style="background: #FFF3A3A6;">ut the initial state contains the key</mark> they are <mark style="background: #FFF3A3A6;">trying to find,</mark> which has now been confused into the keystream and is no longer recoverable.
	- 




