## Stream cipher
- A **stream cipher**<mark style="background: #FFF3A3A6;"> encrypts bits one at a time</mark>, for as long as is necessary. 
- **Conceptually**:
	![](Pasted%20image%2020260127222041.png)
- Each plaintext bit $x_i$ is combined with keystream bit $x_i$ to yield one ciphertext bit $y_i$ 
- Stream <mark style="background: #FFF3A3A6;">ciphers encrypt using **modulo 2**</mark> addition:
	- Let $x, y, z \in \{0, 1\}$
	- $e_{si} = y \equiv x + s_i \ (mod \ 2)$ 
	- $d_{si} = x \equiv y + s_i \ (mod \ 2)$ 
- The<mark style="background: #FFF3A3A6;"> encryption and decryption operations for a stream cipher are identical under this modular arithmetic</mark>, as due to the properties of XOR under binary prime field(s), it behaves as its own inverse. 
	![](Pasted%20image%2020260127222826.png)

### Modulo 2 - XOR
- **XOR** *is a <mark style="background: #FFF3A3A6;">binary operator that returns true if either one of the input is true, but not both*</mark>
- For example: 
	- $x_0, \dots, x_7 = 01010001 \oplus s_0, \dots, s_7 = 10010100$
	- Yields:
		- $11000101$ 
		- We could then invert this and obtain the original by reapplying the key material as the modulo 2 wraps around for the +1. 


### Security of XOR
- In and of itself, <mark style="background: #FFF3A3A6;">XOR provides no security beyond the secrecy of the keystream</mark>
- The <mark style="background: #FFF3A3A6;">**security of a stream cipher** depends entirely on the keystream</mark>
- If the <mark style="background: #FFF3A3A6;">keystream is predictable </mark>then the cipher <mark style="background: #FFF3A3A6;">will be broken</mark>
- If the keystream is **<mark style="background: #FFF3A3A6;">truly random</mark>** then <mark style="background: #FFF3A3A6;">each bit</mark> $x_i$ can be <mark style="background: #FFF3A3A6;">encrypted</mark> to either a <mark style="background: #FFF3A3A6;">0 or 1 </mark>with<mark style="background: #FFF3A3A6;"> equal chance</mark> and the output would not reveal anything about the keystream or input. 
- The p<mark style="background: #FFF3A3A6;">roblem becomes keystream generation</mark>: how can we generate a random, and thus secure, keystream? 


### Randomness
- <mark style="background: #FFF3A3A6;">True randomness</mark> is impossible, to create,  by chance, generated via unpredictable physical processes e.g., thermal noise, radioactive decay, hardware noise such as clock drift, but these are hard to generate and slow to do so
- <mark style="background: #FFF3A3A6;">We approximate randomness</mark> via formulae to generate sequences that look random but are actually deterministic

### Pseudorandom number generators
- A **PRNG** takes:
	- A <mark style="background: #FFF3A3A6;">small **seed</mark>** value
	- <mark style="background: #FFF3A3A6;">Produces long sequence of numbers predicated on seed</mark>
	- Is <mark style="background: #FFF3A3A6;">deterministic</mark>, but<mark style="background: #FFF3A3A6;"> appears random</mark>
- **Linear congruential generator**
	- $s_0 = 12345$ 
	- $s_{i+1} \equiv 1103515245 \cdot s_i + 12345 \ mod \ 2^{32}$ 
		- Predicated on initial seed value, and <mark style="background: #FFF3A3A6;">prior inner state $s_i$</mark> wrapped to 32 bits.
- This is <mark style="background: #FFF3A3A6;">not suitable for cryptography;</mark> <mark style="background: #FFF3A3A6;">given enough output</mark> (<mark style="background: #FFF3A3A6;">ciphertext</mark> and <mark style="background: #FFF3A3A6;">possibly some plaintext</mark>), the <mark style="background: #FFF3A3A6;">internal state can be recovered. </mark>
	- F<mark style="background: #FFF3A3A6;">uture bits become predictable</mark> given they are fo<mark style="background: #FFF3A3A6;">rmed according to prior internal state bits</mark>. 
	- If $s_i$ a<mark style="background: #FFF3A3A6;">nd other key values are known in the generator</mark> (often selected from the literature), <mark style="background: #FFF3A3A6;">all future keystream output</mark> can be predicted
	- Don't need many samples to do this
- <mark style="background: #FFF3A3A6;">We want a PRNG</mark> with the <mark style="background: #FFF3A3A6;">additional requirement</mark> that <mark style="background: #FFF3A3A6;">observing its output</mark> <mark style="background: #FFF3A3A6;">should not be able to allow one</mark> to <mark style="background: #FFF3A3A6;">predict its future output</mark>. 
	- The next bit should not be predictable given all prior output. 

### Cryptographically Secure CSPRNGs
- A **PRNG** <mark style="background: #FFF3A3A6;">with the requirement </mark>that given <mark style="background: #FFF3A3A6;">all previous output</mark> it should be <mark style="background: #FFF3A3A6;">computationally infeasible</mark> to <mark style="background: #FFF3A3A6;">deduce</mark> the <mark style="background: #FFF3A3A6;">next bit</mark>. 
- The <mark style="background: #FFF3A3A6;">output looks random</mark>, and is **unpredictable** and <mark style="background: #FFF3A3A6;">security holds even </mark>if attacker sees a lot of <mark style="background: #FFF3A3A6;">keystream. </mark>
	![](Pasted%20image%2020260127230030.png)
	Answer should be NO for CSPRNG.
- For a **secure** CSPRNG <mark style="background: #FFF3A3A6;">the probability</mark> of <mark style="background: #FFF3A3A6;">guessing the nex</mark>t keystream bit $s_{i+1}$ should be extremely <mark style="background: #FFF3A3A6;">close to 0.5</mark>:$$Pr[x = s_{n+1} < 0.5 + \epsilon]$$
	- Where $\epsilon$ is a<mark style="background: #FFF3A3A6;"> neglibly small advantage</mark> which is determined for the specific CSPRNG. An advantage of even $0.01$ would be far too large and indicates that the CSPRNG is not secure. 



### Unconditional security
> **A cryptosystem is unconditionally secure or information theoretically secure if it cannot be broken even with infinite computational resources.**

### Perfect secrecy
- A **cipher** has <mark style="background: #FFF3A3A6;">perfect secrecy</mark> if the <mark style="background: #FFF3A3A6;">ciphertext</mark> <mark style="background: #FFF3A3A6;">reveals no information</mark> about the plaintext.
- $\forall m_0, m_1 \in M where \ \vert m_0 \vert = \ \vert m_1 \vert \ and \ \forall c \in C$ 
- $Pr[E(k, m_0) = c] = Pr[E(k, m_1) = c]$ 
	- That is, the <mark style="background: #FFF3A3A6;">probability of encrypting</mark> $m_0$ to any given ciphertext $c$ is exactly equal to the probability of encrypting $m_1$ to ciphertext $c$. <mark style="background: #FFF3A3A6;">This means that if the ciphertext is intercepted</mark> it reveals<mark style="background: #FFF3A3A6;"> no information about whether the original input</mark> was $m_0$ or $m_1$ as both inputs are <mark style="background: #FFF3A3A6;">equally as likely to have produced the ciphertext observed</mark>
- Very strong property in aid of **UNCONDITIONAL SECURITY**

### One-time pad
- Does not see practical use anymore
- <mark style="background: #FFF3A3A6;">Truly random keystrream generated </mark>by a **TRNG** same length as the message
- The <mark style="background: #FFF3A3A6;">keystream is only known to communicating parties</mark>
- Every<mark style="background: #FFF3A3A6;"> keystream bit is used exactly once for encryption</mark>, and decryption, respectively. 
- Achieves **perfect secrecy**
	![](Pasted%20image%2020260127232427.png)
	- The key $k$ is chosen uniformly from the set of $K$ keystreams, we say that for given $k, m, c$: what is the probability that given $k$ produces $c$. For a one-time pad, the top half of the formula is $1$ meaning that only one key $k$ could have produced $c$.
	- So always $1/K$, we do not know which key it was.
		- Key was randomly generated too, so will not preserve frequency distributions
- Not practical however; difficult to generate, and 1GB file would need 1GB key, and if a key was reused, the perfect secrecy noted above would be broken. 
	- There is also the issue of key delivery.

## Modern stream ciphers
- <mark style="background: #FFF3A3A6;">Modern stream ciphers</mark> aim to <mark style="background: #FFF3A3A6;">approximate the one time pad</mark>.
- Use an initial seed key (128 bit/256 bit) to generate an *infinite psuedorandom keystream* (typically via a **CSPRNG**) and then use **XOR** exactly like OTP.
- The aim is to approximate $OTP$ whilst ensuring practical usability:
	![](Pasted%20image%2020260127233113.png)
	<mark style="background: #FFF3A3A6;">The difference is</mark>, <mark style="background: #FFF3A3A6;">initialised by seed key via some deterministic construction to get the pseudorandom keystream that cannot have its next bit</mark>(s) $s_{n+1}$ predicted and then $XOR$ to<mark style="background: #FFF3A3A6;"> yield the ciphertext</mark>. The person on the other side has the same $CSPRNG$ and same seed and just xors again to cancel out the keystream. 


### Keystream reuse
- <mark style="background: #FFF3A3A6;">Reusing a keystream</mark> catastrophically breaks a cipher, as they are equivalent, you can XOR the ciphertexts to yield their constituent parts and because that XOR is associative (it is just addition in an integer ring)  the keystream cancels out, giving the XOR of both plaintext messages.
- This happens only by having $C_1$ and $C_2$ 
	![](Pasted%20image%2020260127233749.png)
- They can acquire $M_1 \oplus M_2$ , which does not tell them what the message is, but they are not far off.
- They can determine in fact exactly what positions the messages differ in by the number of 1s in $M_1 \oplus M_2$ 
	- May be able to guess what one of the messages is, depending on the context, or at least part of it, which is unacceptable.

### Crib dragging
- If we have $M_1 \oplus M_2$ from keystream reuse, and suspect $M_1$ encrypts an amount of plaintext that we suspect e.g., ASCII, header
- We can try the following at every position:
	- Position 0: $(M_1 \oplus M_2) \oplus M_p$ = potential $M_2$ substring
		and continue...
- - The reason this works algebraically:
	$M_1 \oplus M_2$ 
	$(M_1 \oplus M_2) \oplus M_1$ 
	$(M_1 \oplus M_1) \oplus M_2$ 
	$0 \oplus M_2$ 
	$\therefore M_2$
- When readable text appears, we know we have found where $M_p$ appears in $M_1$, revealing an equivalent part of $M_2$ 
- Easy to automate on modern PCs and can break stream ciphers.

### Nonce
- <mark style="background: #FFF3A3A6;">Thus, we have to seed based on another random initialiser</mark> to <mark style="background: #FFF3A3A6;">prevent keystream reuse.</mark>
- A **nonce** is an additional seed added to a **CSPRNG**
- This<mark style="background: #FFF3A3A6;"> aids keystream reuse </mark>- <mark style="background: #FFF3A3A6;">one fixed secret key</mark> $k$ is kept, and the <mark style="background: #FFF3A3A6;">keystream is altered depending on the nonce value</mark>
	- Gen diff keystream bits for each message. 
- <mark style="background: #FFF3A3A6;">Nonce usually not secret,</mark> and generated based on set of deterministic rules. 
-<mark style="background: #FFF3A3A6;"> Unique security requirement</mark> is a unique<mark style="background: #FFF3A3A6;"> key+nonce pair,</mark> rather than just unique key; combine key with nonce at encryption/decryption time, rather than just using the plain key, to reduce the potential for keystream reuse. 