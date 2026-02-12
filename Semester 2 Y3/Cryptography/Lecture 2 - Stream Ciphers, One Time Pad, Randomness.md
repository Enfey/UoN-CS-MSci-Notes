## Stream cipher
- A **stream cipher** encrypts bits one at a time, for as long as is necessary. 
- Conceptually:
	![](Pasted%20image%2020260127222041.png)
- Each plaintext bit $x_i$ is combined with one keystream bit $k_i$ to produce one ciphertext bit $y_i$
- Stream ciphers encrypt using **modulo 2 addition**.
	- Let $x, y, s \in \{0, 1\}$ 
	- $e_{s_i}(x_{i}) = y_{i} \equiv x_{i} + s_{i} \ (mod \ 2)$
	- $d_{si}(y_i) = x_i \equiv y_i + s_i \ (mod \ 2)$ 
- The encryption and decryption operations are identical.
	- This is because modulo 2 addition is equivalent to $xor$, which is its own inverse.
		![](Pasted%20image%2020260127222826.png)

### Modulo 2 - $XOR$
- "$XOR$ is a binary operator that returns returns true if either one input or the other is true, but not both"
	![](Pasted%20image%2020260127223103.png)
- For example:
	$x_0, \dots, x_7 = 01010001$   $\oplus$    $s_0, \dots, s_7 = 10010100$ yields:
		$y_0, \dots, y_1$ = $11000101$ 
		We could then invert this given $s$ by applying the same operation to yield $x$. 

#### Security of $XOR$
- $XOR$ in and of itself, provides no security
	- It adds zero resistance beyond the secrecy of $s$
- The security of a stream cipher depends **entirely** on the keystream
- If the keystream is predictable, then the cipher can easily be broken. 
- If the keystream is truly random, each bit $x_i$ can be encrypted to $0$ or $1$ with equal chance, similarly, for some output $y_i$ we could therefore not determine what the original value of $x_i$ was - it could be either.
- The problem thus becomes: how do we generate a random, and thus secure keystream?

## Randomness
- **True randomness** is impossible to recreate except by chance, it is generated via unpredictable physical processes rather than mathematical formulae.
- Computers generally use hardware sources for true randomness:
	- Thermal noise
	- Radioactive decay
	- Hardware noise(clock drift, disk timing, interrupts)
	- These are unpredictable even with infinite computing power, and are extremely strong, but are hard to generate, and often slow to do so. 
- We can approximate randomness via formulae to generate sequences that appear statistically random but are actually deterministic, generally predicated on a seed value. 

### Pseudorandom number generators (PRNGs)
- A **PNRG**
	- Takes a small **seed**
	- Produces a long sequence of numbers
	- Appears random statistically
	- Is actually deterministic
- **Linear Congruential Generator** e.g., C's `rand()`
	- $s_0 = 12345$ $= seed$
	- $s_{i+1} \equiv 1103515245 \cdot s_i + 12345 \ mod \ 2^{32}$ 
- These are not suitable for cryptography; given enough output (ciphertext and possibly some plaintext), the internal state can be recovered. Future bits become predictable given that they are formed according to prior bits. 
	- If they recover the internal state $s_i$ and $a, c$ are known (if not hardening, will have selected standard values from the literature), all future keystream output can be calculated, breaking the keystream as the key can be calculated.
	- Only need a few samples to do this.
- We want a PRNG with an additional requirement that the next bit should not be predictable given all previous output. 

### Cryptographically Secure PRNGs (CSPRNGs)
- A **PRNG** with an extra requirement; given all previous output, it should be computationally infeasible to predict the next bit.
- The output looks random, output is **unpredictable**, and security holds even if the attack sees a lot of the keystream.
	![](Pasted%20image%2020260127230030.png)
	For a CSPRNG, the answer should be no.
- For a secure CSPRNG, the probability of correctly predicting the next bit should be extremely close to 0.5$$Pr[x = s_{n+1} < 0.5 + \epsilon]$$Where $\epsilon$ is a negligibly small advantage. This advantage is according to the CSPRNG, an advantage of 0.01% would be far too large, and indicate that the PRNG is not cryptographically secure.

### Unconditional Security
> A cryptosystem is **unconditionally or informational-theoretically secure** if it cannot be broken even with infinite computational resources.

### Perfect Secrecy
> A cipher has **perfect secrecy** if the ciphertext reveals no information about the plaintext.
- $\forall m_{0}, m_{1} \in M \ where \ |m_0| = |m_1| \ and \ \forall c \in C$ 
- $Pr[E(k, m_0) = c] = Pr[E(k, m_1) = c]$ 
	- Where $M$ is the message space, and $m_i$ is a message in the message space.
	- The probability that encrypting $m_0$ produces ciphertext $c$ is exactly equal to the probability that encrypting $m_1$ produces that same ciphertext $c$. This means that if a ciphertext is intercepted, there is zero information gained about whether the original message was $m_0$ or $m_1$, every possible plaintext is equally likely to give the ciphertext observed. 
	- This is an incredibly strong security property in aid of **unconditional security.**

### One-time pad
- Does not see practical use anymore, was used by spies in the cold war.
- Truly random key stream $s_0, s_1, s_2$ generated by a **TRNG**, exactly the same length as the message. 
- The key stream is known only to the communicating parties
- Every key stream bit is **used exactly once,** for encryption and decryption
- Achieves **perfect secrecy**.
	![](Pasted%20image%2020260127232427.png)
	- The Probability formula is that way because the key $k$ is chosen uniformly at random from $K$ (the set of all keystreams). We essentially say that for given $k$, $m$, $c$, what is the probability that given $k$ produces $c$, $m \ \oplus \ k = c$ . For a one-time pad, the top half of the formula is $1$ meaning that exactly one key uniquely yields $c$, and no other key could derive the same $c$ for $m$. This is because...
	- So always 1/K, we don't know which key it was
	- Frequency analysis wouldn't work as the key was randomly generated, and was truly random, it will not preserve frequency distributions. 
- A one time pad is not practical
	- A 1GB file would need a 1GB key
	- If a key is ever reused, the entire cipher can be broken. 
	- Difficult to distribute too. 

### Modern Stream Ciphers
- Modern stream ciphers replaces the **one time pad**
- Uses an initial seed key (128 or 256 bit) to generate an infinite pseudorandom keystream, usually via a **CSPRNG**, and then use $XOR$ exactly like **OTP.**
- Aim is to approximate OTP, ensuring practical usability. 
	![](Pasted%20image%2020260127233113.png)
	The difference is, initialised by the seed key to generate the infinite pseudorandom keystream, and then $XOR$ to get $y$, and the person on the other side has the same algorithm and the same seed key, and $XOR$s the ciphertext y to yield x. 

#### Keystream reuse
- ![](Pasted%20image%2020260127233749.png) 
- Even though the attacker only has $C_1$ and $C_2$, they can acquire $M_1 \oplus M_{2}$ as anything $XOR$'d with itself is $0$, so the key becomes irrelevant $(K \oplus K = 0)$ . 
- This does not tell us what the message is, but we are not far off. 
- The attacker can determine which positions the messages differ in exactly via this result. 
- ![](Pasted%20image%2020260127233922.png)
- That is, the final 2 bits. The attacker may be able to guess, via patterns in this output, what the message is. E.g., is anything is 0, then identical. 

#### Crib Dragging
- Say we have $M_1 \oplus M_2$ from the keystream reuse. We suspect that $M_1$encrypts an amount of plaintext that we know about or suspect.
	- An example text could be an a header, ASCII text, or the first part of an english word.
- We can try this at every position:
	- Position $0$: $(M_1 \oplus M_2) \oplus txt$  = potential $M_2$ substring
	- Position $1$: $(M_1 \oplus M_2)[1:] \oplus txt$  = potential $M_2$ substring ![](Pasted%20image%2020260209235924.png)
		In this scenario, we try 2 bytes of predicted M_1 plaintext.
- When readable text is uncovered in $M_2$ , we have found where $txt$ appears in $M_2$.
- The reason this works algebraically:
	$M_1 \oplus M_2$ 
	$(M_1 \oplus M_2) \oplus M_1$ 
	$(M_1 \oplus M_1) \oplus M_2$ 
	$0 \oplus M_2$ 
	$\therefore M_2$
- This is easy to automate on modern PC's, and can break stream ciphers; at the very least uncovering some bytes of $M_2$. The fact that any part can be decrypted by an attacker breaks the security of the encryption system e.g., could be the digital signature verifying the bank transfer. 

#### Modern stream ciphers - Nonce
- We have to seed not just based on our key, but based on another random initialiser
- A $nonce$ is an additional seed that's added to a **CSPRNG.**
- This aids keystream reuse problems - one fixed secret key $k$ is kept, and the keystream is altered via the nonce value for key $k$.
	- Generate different keystream bits for each message, rather than generating a new key as input to the CSPRNG.
	- Less key generation, and hardens against keystream reuse attacks as changes for every message.
		- Otherwise, could break sequential messages.
- A nonce is not secret. 
	- Usually generated based on a set of deterministic rules
	- In TLS, use a counter, and reserve some bits in the message so the other person knows which one you used. ![](Pasted%20image%2020260210001933.png)
- Nonces are thus vital for stream cipher security.
- Instead of always using a unique key, the security requirement is now that you always use a unique $(key + nonce)$ pair.






