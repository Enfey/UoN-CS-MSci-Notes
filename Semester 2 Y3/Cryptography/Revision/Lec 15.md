## Hash functions
- Could split and sign each part of message independently
	- Problematic, size of sigamnature grows linearly with message size; in real world message size= arbitrary
	- Each block is independently signed, attacker could reorder blocks of a message and signatures would still verify correctly, no way to determine which ordert the original message was sent in. 
- Hash tghen sign
	- Preferred, modulus, prevent malleability, stay under modulus, and prevent existential modulus
	- Fixed size digest stays under modulus
	- Has determinism 
- Must design hash funciton to yield good properties to make possiblity of attacks less likely. 



## Properties of Hash functions
1. **Any input length**
2. **Fixed output length**
3. **Preimage resistance**
4. **Second pre-image resistance**
5. **Collision resistance**

### Input and Output length
- Hash function maps from  $\{0, 1\}^* \to \{0, 1\}^n$
- The message space is infinite, whilst the message space is finite
- Of course, this tells us immediately that collisions must exist via the pigeonhole principle, infinitely many inputs must map to the same output
- It should however, bec omputationally infeasible to find said collisions. 

### Preimage resistance
- Given $H(x)$, it must be computationally infeasible to **invert** and find x
- Hash functions are non-invertible.
- Less applicable to digital signaturs, it does help for preventing things like existential forgeries, but is more important for things like password storage
	- Must not be able to invert the hash and compute the original message.

### Second Preimage Reisstance
- Given a message $x_1$ and a hash $H(x_1)$ it should be infeasible to find another message $x_2$ such that $H(x_1) = H(x_2)$ where $x_1 \neq x_2$ 
- This is essential for digitial signatures, if someone can find a message that has the same hash as a prior transmitted message, then their signature (except under the most advanced schemes e.g., probabilistic signaures) will also be identical
- The existing signature $s$ can therefore be used to validate the new message, letting attackers send the message with verification without needing the private key.
- Finding this by brute force for a 256 bit hash with good random properties requires $2^{256}$ attempts
- The attack is detailed below:
	- Literally just swap out the message while its in transit LOL.



### Strong Collision Resistance
- It must be infeasible to find **any pair** $x1, x2$ such that their hashes are equivalent
	- The message is not fixed.
- This is distinct from the above,. the message is not fixed. 
- In practice, this is much easier to break. 
- In the attack scenario, the MITM has control over the message:
	- Invisible modifications are made to both messages e.g., extra space, different encodings, etc computing hashes of both until they match
	- When they match, send the original innocent message which is then signed by Bob, and send to Alice
	- Then swap out the message while in transit to present the new message along with a valid signature, which is verified by Alice.


#### Birthday Paradox and Collision Attack
- The attacker just needs two messages that collide with each other. 
- The **birthday paradox applies**
	- What is the probability two people in this room share a birthday
- It is easier to first compute the probability $P(n)$ that $n$ people do not share a birthday
	![](Pasted%20image%2020260329234940.png)
- For a hash function with $2^n$ outpouts, the probability of a collision among $k$ hashes is ![](Pasted%20image%2020260329235241.png)
- We find a collision of approximately find a collision after approximately $\sqrt(2^n) = 2^{\frac{n}{2}}$ random attempts 
- That means the bit length output of $H$ needs to be double the size of the desired security margin


### Merkle-Damgard
- Use compression function to build a collision resistant hash function. 
- The message is split into fixed-size blocks and processed sequentially
- Compression function takes: $x_{i}$ and $H_{i-1}$ 
	- This then produces $H(x_{i})$ 
- The key theoretical result is if the compression function $f$ is collision resistant for fixed size inputs, then the entire Merkle-Damgard construction is actually collision reistant for arbitrary length inputs
	- The security is tied to the compression function. 

### SHA-256
- Concrete instantion of Merkle-Damgard
- Before any hashing, message padded to multiple of 512 bits
	![](Pasted%20image%2020260330031702.png)
- The message $x$
- The single $1$ to mark where message ends
- Zero padding fills space so total reaches multiple of 512 bits $y \equiv 512 - l - 1 - 64 \mod 512$ if result would be negative, then padding spills over into a completely new block
- 64 bit length field $l$ encodes the original message length in bits, prevents length-extension attacks. 


### Message schedule
Split into 16 words, enough for 16 rounds
Expand to 64 words
Compute words 16-63,c omputed from previour words, use right shifts to LOSE information, non-invertible function. Use of 3 differewnt source words to compute one means that their contents diffuse and meld together.


Round function uses a round constant, 1 per round, break symmetry even when the opeation is identical. 4 operations per round, each providing either confusion or diffusion. 