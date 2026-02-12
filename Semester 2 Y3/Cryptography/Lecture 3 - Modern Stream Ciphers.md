# Modern stream ciphers
- Aim to approximate the **one-time pad**
- Use an initial fixed-length (128 or 256 bit ) seed key to generate an infinite pseudorandom keystream, via a **CSPRNG**, and then use $XOR$ exactly like $OTP$
- Practically usable. 
	![](Pasted%20image%2020260127233113.png)
	Initialised by the seed key to generate the infinite pseudorandom keystream, and then $XOR$ to get $y$, and the person on the other side has the same algorithm and the same seed key, and $XOR$s the ciphertext y to yield x. 
## Psuedorandomness
- There are three types of random number generators:
	- **True Random Number Generators (TRNGs)**
	- **Psuedorandom Number Generators (PRNGs)**
	- **Cryptographically Secure Psuedorandom Number Generators(CSPRNGs)**
- **TRNGs** are impractical at scale and slow
- **PRNGs** are fast and deterministic, but predictable given enough output; internal state can be recovered, and future keystream bits can be determined to break the cipher. 
- **CSPRNGs** are deterministic but *computationally unpredictable*. Given outputs, a well designed CSPRNG means that an attacker cannot recover internal state, past outputs, or future outputs. Modern stream ciphers are these, designed such that the keystream is computationally indistinguishable from true randomness. 

## Flip-flops
- We briefly recap flip-flops as early stream ciphers were hardware-oriented and clock driven. 
- $D$ **flip-flops** capture the value of the $D-input$ at a definite portion of the clock cycle, typically the rising edge of the clock. 
	- Simply reflect the input into the output based on the clock pulse.
		![](Pasted%20image%2020260210180339.png)
- This captured values becomes the $Q$ output. At other times, the output $Q$ does not change. 
- Holds a single bit of information.
- However, can set up a circuit of these flip flops in the form of **linear feedback shift registers**, to product statistically random looking data.

| Clock       | $D$ | $Q_{next}$ |
| ----------- | --- | ---------- |
| Rising edge | 0   | 0          |
| Rising edge | 1   | 1          |
| Non-rising  | X   | $Q$        |

## Linear feedback shift registers
- **A linear feedback shift register** is an **$m$-bit** register, that shifts bits to the right every clock cycle, and computes a new input bit (known as **feedback**) to the $LSFR$ as the $XOR$ of some bits of the overall shift register value.
	D-input comes from the circuit and from the feedback
- Usually comprised of flip-flops, the last bit represents the output.
- Clocking LSFRs generates a sequence of output bits.
	![](Pasted%20image%2020260210181558.png)
- We can make this more complex; we are not limited by the number of flip-flops, or feedback complexity.
	![](Pasted%20image%2020260210181929.png)
	Ignore the 1000, left in by mistake.
- These LFSR's are bad; we added another XOR gate, but imagine we're using 128 bit LFSR to produce interesting random data, but actually only have a sequence length of 10, and that is predictable, not interesting statistically.
- Must design a good LFSR.

### Mathematical representation of LSFRs
![](Pasted%20image%2020260210185909.png)
- This is the most general form of an LSFR
- We have $m$ **flip-flops**; an $m$ bit LSFR.
	$s_{m-1}, s_{m-2}, ..., s_1, s_0$ 
- At time $t$ its internal state is $s_t = (s_t^0, s_t^1, ..., s_t^{m-1}) \in \{0,1\}^m$ 
	This update is deterministic. 
- The rightmost bit $s_0$ is the output bit. 
- Selected bits are fed back through XOR gates to form the new input bit. 
- Everything happens at once on a clock edge, we do not shift and then feed back later. 
- The symbols $p_0, p_1, ..., p_{m-1}$ are called taps
	Each $p_i \in \{0, 1\}$, if $p_i = 1$ then that register contributes to feedback, if $p_i = 0$ then that register is ignored. The symbol is multiply
- The equations describe how each new input bit is a **linear combination** of previous bits
	Everrything shifts for the second equation, so $s_m$ replaces $s_{m-1}$ etc, but the taps stay fixed.
	Because it is a linear equation, this provides an attack vector. More on that further on. 
#### Polynomial representation of LSFRs
- We usually represent $m$ **bit** **LSFRs** using polynomials of degree $m$.$$P(x) = x^m + p_{m-1}x^{m-1} + \dots + p_{1}x + p_{0}$$
- Degree $m$ = number of flip-flops
- Coefficients $p_i$ = which taps are enabled
- This is essentially shorthand for the architecture of the circuit. 

##### Example
$$P(x) = x^3 + x + 1$$
	![](Pasted%20image%2020260210191528.png)

### Maximum length LSFRs
- "When is an LSFR good"?
- An m-bit LSFR can have at most $2^m-1$ states.
	The all zero state loops forever, so it is excluded.
- An LSFR that achieves this maximum is called a **maximum-length LSFR**.
	![](Pasted%20image%2020260210191820.png)
	Are some examples of LSFRs that have **primitive polynomials** - the key theorem is that an LSFR produces a maximum length sequence iff its feedback polynomial is primitive. This is outside of the scope of the module however.
- LSFRs period is the number of steps before the internal state repeats. 
	- Thus, the period is directly tied to the number of cycles before a keystream begins repeating itself. 

### Attacking LSFRs
- Assuming a **known plaintext attack**, attacker knows $2^m$  plaintext bits e.g., know HTTP request, so have the header
- Attacker has the ciphertext
- XOR, can retrieve some keystream bits $c = m \oplus k$, $k = m \oplus c$ 
	1. Calculate the key bits
		$s_i \equiv y_i + x_i (mod \ 2), i = 0, 1, ..., 2_{m-1}$ 
	2. Reconstruct the LSFR
		$s_m = s_{m-1}p_m-1 + ... s_1p_1 + s_0p_0 (mod \ 2)$ 
		$s_{m+1}$...
		Each bit gives a linear equation, unknowns are the $p_i$ coefficients. This means that the internal state can be recovered, so all future output can be predicted. 
		Do have to guess at size of LSFR. 
- LSFRs are linear systems, that can be written as such linear equations, and solved, given keystream bits. 
	- Next state = linear function of the current state ($XOR$ is linear over $GF(2)$), if remain purely linear over $GF(2)$, then able to predict keystream. 
- Therefore no single LSFR is cryptographically secure, no matter how long. 
	Also cannot use them natively, have to put them into things?
## Trivium
- The solution to LSFRs linearity is to combine multiple LSFRs where we have some nonlinear combination of their outputs as the true output. 
	We may use an AND gate, as this is a non-linear function over $GF(2)$
- Unbroken stream cipher based around the idea of LSFRs.
- In **trivium**
	- 3 **LSFRs**
		- 93 bits
		- 84 bits
		- 111 bits
	- Chosen to have incompatible periods
		- That is, they have no shared large common divisors (aim for $GCD$ = 1), so the LCM of their periods is roughly equivalent to their factor
			$LCM \approx (2^{93} - 1)(2^{84}-1)(2^{111}-1)$
		- The reason for this is that say if the periods align, the period of the overall system would be lower, and there are less distinct internal states reachable before repetition. 
		- Helps prevent statistical biases.
	- Feedback each
	- Non-linear AND gates in feedback
	- Trivium uses an 80-bit key, and 80-bit nonce as initialisation
	- We cannot represent this system as a linear combination of bits, and so cannot represent as linear equations, so we cannot solve it.
## Chacha20
- Asynchronous stream cipher
- XOR, Add, Rotate, 
- 4 constants
- 256 bit 8 key values
- Block number, and nonce (can extend or increase these 2, TLS increases nonce and nonce+keystream reuse is catastrophic).
- We add, so that it is non-invertible, after the round function. 
- Round function
	- need to be extremely good at mixing input, e.g., if didn't do anything, could just divide by 2 to acquire input after add and derive the Key. 
	- need to be robust against weak keys also.
		- This is what constant values provide. 
	- Quarter rounds operating on 1/4 of the block, alternate column and diagonal rounds.
	- Propagation/diffusion.
![](Pasted%20image%2020260210202800.png)

