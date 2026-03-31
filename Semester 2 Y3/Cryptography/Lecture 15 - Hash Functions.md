## Why Hash Functions
- We refer back to digital signatures with a simple and important question: what if the message $m$ is too long to sign directly?
- Two naive approaches are considered:
	1. Split and sign each part of the message independently.
		- $s_0 = sig(x_0), s_1 = sig(x_1), ... s_n = sig(x_n)$ 
		- This is problematic because the size of the signature grows linearly with the message size, making verification and signing more computationally intensive for larger documents.
		- Each block is independently signed, so an attacker could reorder blocks of the message or the signatures and each would still verify correctly, in theory, and as such there is no integrity over the message as a whiole
	2. Hash then sign
		- $z = H(m)$ and then $s = sig(z)$, compresses arbitrarily ,long messages to a fixed-sized digest
		- Solves both of the above problems in the prior approach
- The reason we have a limitation of message size is that a message $m$must be $> n$, the modulus
- This is because a value greater than the modulus would have an equivalent under that modulus, which would make decryption impossible, as it would simply resolve to that equivalent rather than the intended content.
	- We need determinism.
	- Arithmetic happens in the ring $\mathbb{Z}_n$ modulo $n$ for RSA
- A hash that produces a fixed-size output over $M$ (with less bits than the RSA keys) will always fall under this constraint.

## Properties of Hash Functions
1. **Any input length**
2. **Fixed output length**
3. **Preimage resistance**
4. **Second preimage resistance**
5. **Collision resistance**


### Input and Output length
- A hash function maps $\{0, 1\}^* \to \{0, 1\}^n$
- The message space is infinite, whilst the output space is finite.
- This principle immediately tells us that collisions must exist via the pigeonhole principle, infinitely many inputs must map to the same output.
- The goal is never to eliminate collisions, but to make them computationally infeasible to find. 

### Preimage resistance
- Given $H(x)$ it must be computationally infeasible to invert and find $x$
	![](Pasted%20image%2020260329231638.png)
- This is less applicable to digital signatures (it does partially help in preventing existential forgeries, see prior lecture) but is extremely important for things like password storage.
	- Must be not able to invert the hash and compute the original input.

### Second Preimage Resistance
- Given a message $x_1$ and a message $H(x_1)$ it should be infeasible to find any $x_2 \neq x_1$ such that: $$H(x_{1}) = H(x_{2})$$ 
	![](Pasted%20image%2020260329231938.png)
- This is essential for digital signature integrity.
- If someone can find a message that has the same hash as a prior transmitted message that has been signed, then their signature (except under the most advanced schemes) will also be identical.
- The existing signature $s$ can therefore be used to validate the new message $x_2$, letting attackers send messages without ever needing the private key or work in reverse.
- Finding this by brute force for a 256 bit hash with good random properties requires approximately $2^{256}$ attempts.
- The attack is detailed below:
	![](Pasted%20image%2020260329232319.png)
	MITM simply swaps out the message contents with their message that has the same hash; when applying the public RSA key to the signature $s$ it will yield the same hash $z$ that would've been yielded for the original message contents.

### Strong Collision Resistance
- It must be infeasible to find **any pair** $x_1, x_2$ such that $H(x_1) = H(x_2)$, with no constraints on either message.
- This is distinct from second preimage resistance because there is no fixed $x_1$ target to find, this is just any two messages that collide with each other.
- In practice, this is much easier to break.
- In the attack scenario, the MITM now has control over the message, and can freely modify both before anything is signed
	- Invisible modifications are made to both messages e.g., extra space, different encodings, etc computing hashes of both until they match. 
	- When they match, send the original, innocent message $x_1$ which is then signed by bob's private key, and then sent to Alice to verify.
	- Oscar discards $x_1$ and presents $(x_2, s)$ with the malicious message, which is then verified by Alice and processed because it has the same hash, so $s^e \ mod \ n \equiv H(x^2)$, even though $s$ was derived from $x_1$ 
		![](Pasted%20image%2020260329235949.png)

#### Birthday Paradox and Collision Attack
- The attacker just needs two messages that collide with each other, without fixing a particular message. 
	- There are many more collisions beyond those simply with $x_1$ when the message space is infinite.
- The birthday paradox applies:
	*What is the probability two people in this room share a birthday.*
- It is easier to first compute the probability $P(n)$ that $n$ people do not share a birthday.
	![](Pasted%20image%2020260329234940.png)
	The probability of at least one collision is $1-P(no collision)$ 
	The probability of a collision with only 23 people is roughly 50%
	The number of pairs to check is:
		![](Pasted%20image%2020260329235123.png)
		253 chances is quite a lot, with each pair having a 1/365 chance of matching!
- This feels wrong, we instinctively think about the probability of someone matching our own birthday specifically, which would indeed require around 183 people for 50%. But we ask whether anyone matches anyone.
- For a hash function with $2^n$ possible outputs, the probability of a collision among $k$ computed hashes follows exactly the same mathematics as the birthday problem:
	![](Pasted%20image%2020260329235241.png)
- We find a collision after approximately $\sqrt(2^n) = 2^{\frac{n}{2}}$ random attempts
- That means the bit length output of $H$ needs to be double the size of the desired security margin
- SHA-256 therefore offers equivalent security to AES-128.


## Merkle-Damgard
- Hash functions need to accept arbitrarily long inputs but produce a fixed sized output. 
- A merkle damgard construction is a method used to build a collision-resistant hash function from a smaller component called a **compression function**
- It is the design principle behind many popular hashing algorithms. 
- The message $x = \{x_1, x_2, \dots , x_{n}\}$ is split into **fixed-size blocks** and processed **sequentially**
- A compression function $f$ takes two inputs:
	- The current message block $x_i$ 
	- The previous hash state $H_{i-1}$ 
- This function then produces the next hash state $H_i$. The final state after all blocks is the hash output $H(x)$ 
- The key theoretical result is if the compression function $f$ is collision resistant for fixed-size inputs, then the entire Merkel-Damgard construction is collision resistant for arbitrary length inputs.
	- This means if you want to break the hashing algorithm, you must break the compression function. 
	![](Pasted%20image%2020260330031417.png)

## SHA-256
- Concrete instantiation of Merkle-Damgard with specific parameters.
	- **Message block size**: 512 bits
	- **Internal state size**: 256 bits
	- **Output size**: 256 bits
	- **Rounds per message block**: 64
- Before any hashing begins, the message must be padded to a multiple of 512 bits. The padding structure is:
	 ![](Pasted%20image%2020260330031702.png)
	- The message $x$: $l$ bits long
	- The single $1$ bit: marks where the message ends. 
	- The zero padding fills space so the total reaches a multiple of 512 bits, length determined by this formula $y \equiv 512 - l - 1 - 64 \mod 512$. If the result would be negative and there would not be enough space, then the padding spills over into a completely new block. 
		- Need at least 65 bits of padding (1 terminator, zero zeroes, 64 bits describing $l$ of $x$)
		- So anything close to $512$ will wrap around and a new block containing 448 zeroes (maybe the separator bit of the final block was exactly 512) and $l$.
	- The 64 bit length field: Encodes $l$, the original message length in bits. This prevents length-extension attacks, discussed next lecture.
	- ONLY THE FINAL BLOCK CONTAINS THE SEPARATOR
- The compression function mixes 512 bit blocks of the message into the current state $H_{i-1}$ to produce a new state $H_i$ 
- It achieves this through two stages: the **message schedule** and **64 rounds**


### Message Schedule
- The compression function runs 64 rounds, and each round needs a fresh 32-bit value from the message to mix in. 
- The message block is only 16 x 32 bit words, enough for 16 rounds
- The message schedule expands these 16 words into 64 words for each round. 
- The 512-bit message block $x_i$ is simply split into sixteen 32 bit words; words 0 to 15
- Words 16 to 63 are computed from previous words:$$W_{i} = W_{i-16} + \sigma_{0}(W_{i-15}) + W_{i-7}+\sigma_{1}(W_{i-2}) $$
	- Pick 2 older words, and apply a rotation and shift to the more recent of them
	- Pick a middle word and a recent word, and apply rotations and shifts to that
- The $\sigma$ functions are:
	- $\sigma_0(W) = (W >>> 7) \oplus (W >>> 18) \oplus (W >> 3)$ 
	- $\sigma_1(W) = (W >>> 17) \oplus (W >>> 19) \oplus (W >> 10)$ 
	$>>>$ is the circular right rotation
	$>>$ is the logical right shift.
	$\oplus$ is the xor.
	- We use the rotation to spread every bit of the word across multiple positions. Rotating by different amounts and then XORing means every bit of the input influences many different bits of the output. 
	- The logical shifts introduce asymmetry; it is not reversible because information is lost by introducing zeroes. Contributes to the non-invertible property. 
	- Use of four different source words means each expanded word depends on a spread of previous words.
- The net effect is that the message schedule coming into the round function is a pseudorandom expansion of the original message, but the messages are completely determined equally by the original 16 words.
### Round Function
- The 256 bit state $H_{i-1}$ is split into eight 32-bit variables named: $a, b, c, d, e, f, g, h$ 
- These eight variables are what are transformed across the 64 rounds to produce $H_{i}$ 
- Each round takes the current $a$ through $h$, mixes in one message word $W_i$ and one round constant $K_i$ and produces a new $a$ through $h$.
- The round constants $K_i$ 
	- 64 fixed constants, one per round. These are derived from the fractional parts of the cube roots of the first 64 prime numbers.
	- For example $^3\sqrt3{2} = 1.2599210 ... \to 0.2599210... \to K_0 = 0x428a2f98$
	- Their purpose is to break symmetry between individual rounds on a particular block of message; without them, all 64 rounds would perform relatively identical operations, making the structure much weaker.
- There are 4 operations per round:
	- **Choose** $$Ch(e, f, g) = (e \wedge f) \oplus (\neg e \wedge g)$$
		- For each bit position independently:
			- If the bit of $e$ is $1$, then take the bit from $f$
				- $1 \wedge f = f$ 
				- $\neg 1 \wedge g = 0$ 
				- $Ch = f \oplus 0$ 
			- If the bit of $e$ is 0, then take the bit from $g$
				- $\neg 0 \wedge g = g$ 
				- $0 \wedge f = 0$ 
				- $Ch = 0 \oplus g = g$ 
			![144](Pasted%20image%2020260330205526.png)
			The formula is provably a bit selector in all cases.
			The output depends on the value of $e$ in a way that cannot be expressed as a simple linear combination. 
	- **Majority** $$Maj(a, b, c) = (a \wedge b) \oplus (a \wedge c) \oplus (b \wedge c)$$
		- For each bit position, outputs whichever value appears in at least two of the three inputs.
			![](Pasted%20image%2020260330210045.png)
		- Also nonlinear, as the output depends on combinations of three inputs that cannot be easily expressed.
	- $\sum_0$ and $\sum_1$ 
		- ![](Pasted%20image%2020260330210231.png)
		- Rotate their input by three different amounts and XOR the results
		- The purpose is to provide diffusion whilst the other two steps provide confusion. 
		- Rotating by three different amounts which are spread far apart ensures that every bit $a$ influences many different output bits, and then $XOR'ing$ those results ensures full influence.


![](Pasted%20image%2020260330210131.png)
- Note that in future rounds, the newly computed $a$ and $e$ will diffuse across the state due to the shift of values at the end of the round.
	- Such that the $H_i$ yielded at the end of round 63 is sufficiently diffused with the different values computed by the diffusion+confusion round functions. 
- After round 63, the output is added back to the original input state to that round $H_{i-1}$
	- This is a key part of what makes this construction non-invertible.
	- Even if you were able to run the rounds backwards, you would need to know $H_{i-1}$ to undo the addition. 
- The state is updated as follows:
	- $T_1 = h + \sum_1(e) + Ch(e, f, g) + K_i + W_i$ 
	- $T_2 = \sum_0(a) + Maj(a, b, c)$ 
	![](Pasted%20image%2020260330211325.png)

 

### $H_0$ 
- The very first block has no previous state $H_{i-1}$ to use.
	![](Pasted%20image%2020260330210720.png)
- SHA-256 defines fixed starting values for $a$ through $h$:
	![](Pasted%20image%2020260330210735.png)
- These are derived from the **fractional parts of the square roots of the first 8 prime numbers**


