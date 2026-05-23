## Signatures and Message auth
- Compute **signature** over a message to give some authenticity to it and its contents, usually that it has not been tampered with. 
- $sig_k(x) = y$ 
- The computed binary signature is appended to the respective document
-  Verification asserts: $$(x, y) \rightarrow \mathrm{ver}_k(x, y) =
\begin{cases}
\text{True;} & y \text{ is valid} \\
\text{False;} & y \text{ is invalid}
\end{cases}$$
- The signature provides a few properties and namely does not provide another:
	- **Authenticity**
		- Only someone with the key $k$ could have produced the signature, where $sig$ is an injection(one-to-one, map distinct messages onto unique co-domain elements, but not all of them, so not bijection) for a key $k$ and message $x$ 
		![](Pasted%20image%2020260329182009.png)
	- **Integrity**
		- The signature is tied to the exact content of the message; if the message is altered, the signature will be invalid as reverifying the message will result in invalid $y$ 
	- **Non-repudiation**
		- Assurance that a party in a transaction cannot deny their authenticity or actions
		- Sig predicated on key $k$, (where here, we pretend the key is symmetric)
		- There would be no non-repudiation because either party could have generated the signature, with no way to attribute who sent it. Alice could deny sending a message by claiming Bob forged it. 
		- We do not have this properrty without the use of public keys



### Public Key Signatures
- Asymmetric cryptography grants us non-repudiation with digital signatures
- Alice has private key and public key
- Alice signs with her private key, and bob verifies with her public key, via the verification algorithm

#### RSA signaturs
- Setup is as follows:
	- Alice's private key: k_prv(d)
	- Alice's public key: k_pub(n, e)
	- Signing: sig_{prvA}(x) = x^d mod n
	- Send (x, s) to bob for verification, along with public key
	- check x' = x
- The efficiency of signing and verification is governed by the use of the square and multiply algorithm, in practice e is kept small, with a good binary representation to minimise either of the operations whilst allowing large enough primes to be chosen
- This prioritises verification speed over signing however, but that is fine because we are usually verifying, not signing. 


## Signature forgeries
- Used x so far, now use m
- A forgery is a valid $(x, m)$ pair for a message $m$ that the legitimate signer did not sign
	- Replkay doesn't count as it was actually signed
- There are various severities of attack classes depending on the degree of control over the message $m$


### Existential Forgeries
- The attacker can create a valid m, s with no ability to select m's contents
- Given a public key, the attacker
	1. Selects random $s$ 
	2. Computes $m' = s^e \ mod \ n$ 
	3. . Output $(m', s)$ as a forged pair.
- $s^e \ mod \ n$ is used to recover $m'$ definitionally.
- Says nothing about validity; it is indeed a valid pair because we have no original message to check against, completely synthetic. 
- Not very useful, but not completely harmless
- m' is essentially a random number with no semantic meaning, and the attacker has zero control over what m' says. 
### Selective Forgeries
- Here the attacker fixes $m$ first and attempts to find a valid signature $s$ that corresponds to $m$
- It is a requirement that $m$ is fixed/**chosen**, otherwise would just be existential forgery
- Computing a selective forgery is equivalent to computing the actual signature under the private key, whhich without, would need to factor n to get the totient, computationally infeasible.
- Selective forgery is closely related to **chosen message attacks** where an attacker can query a signing oracle for signatures on chosen messages. 
	- The question then becomes whether seeing a valid signature for arbitrary messages helps forge a signature on the target $m$; do they reveal anything about $d$?
	- Believed to be difficult.


### Universal forgery
- The attacker is able to sign any message $m$ on demand and compute a corresponding signature $s$ yielding a valid m/s pair.
- For RSA almost always means they have recovered $d$; only way to produce valid signatures under a key for arbitrary messages. 
- Equivalent to completely breaking RSA and subsumes the other 2 levels. 

## Malleability
- A cryptosystem is deemed **malleable** if the attacker can modify the ciphertext or signature in a meaningful way and produce another valid ciphertext/signature predictably. 
- RSA signing is **modular exponentiation**
- **Exponentiation distributes over multiplication.**
	- This means that one can produce a valid signature for the multiplication of messags by simply multiplying their signatures.
	- This means you can transform a ciphertext/signature under RSA into another valid signature without knowing the key directly.
	- This is more control than we would like an attacker to have over a signature scheme
	- m3 is random and will not yield a meaningful transaction, this is just another way to achieve existential forgery.

### Padding
- The solution to the aforementioned weaknesses, particularly malleability and existential forgery is to enforce strict formatting on $m$ prior to signing. 
- If valid messages must follow a specific structure, then randomly generated forged messages from either malleability or from applying e to a message to achieve existential forgery and unlikely to conform to this structure; the message is not permitted to be random. 
- We include a $y$ bit padding field for every message $m$
	![](Pasted%20image%2020260329194808.png)
- For a generated message $m' = s^e \ mod \ n$ it must contain all valid $y$ bits to be treated as a proper message. 
- The probabiliity that any $bit_y$ is a correct bit under a brute force or extenstial forgery is $1/2_y$, or $2^{-y}$ 


### Hash then sign
- Signer computes non-invertible $PRF$ over the message called a hash
- :$$sig_{k_{prvA}}(m) \equiv H(m)^d \ mod \ n$$
- This is then recomputed by the verifier by applying the public key to the message, and checking the hashes are equivalent, rather than direct message contents

#### Benefits
- MEssage size
	- Raw RSA can only sign values smaller than $n$
	- This is because values greater than it, under the modulus would have an equivalent and would wrap back around, making decryptionimpossible as a message could be one of a few variants
	- Real messages are arbitraroily large
	- Hashing compresses to fixed-digest, ensuring message space compressed to $<n$ 
- **Existential forgeries become impossible**
	- Signatures are now computed over a hash
	- For an existential forgery an attacker takes a random signature and applies $e$ to it with no control over the message
	- An attacker has $s^e$ mod n  computed over the hash
	- But they do not have the message this hash belongs to
	- Signatures now correspond to a hash, rather than a message, to be able to forge a signature and provide a complete message, they would have to invert the hash function.
		- Hash functions are **preimage resistant, meaning that given a hash, finding a message that produces it is impossible.**
- **Malleability**
	- RSA is no longer computed over the messages directly, so the exponentiation distribtuting over multiplication property that permits one to forge signatures for a multiplication of a message, is now infeasible, as RSA is computed over the hashes, not eh messages
	- Finding messages whose hash satisfies this relationship is infeasible for a good hash function


### PKCSv1.5
- Standard that defines a precise encoding for the padded message before signing
	![](Pasted%20image%2020260329202953.png)
- First component = 0x00, ensure encoded message is numerically less than $n$
- The second component signals this is a private key operation, $0x02$ used for encryption. 
- $0xFF..FF$ padding string, must be at least 8 bytes of $0xFF$
	- Adds constrained bits to reduce forgery probability
	- Ensures total length of message $m$ prior to signing is less than the modulus size.
- 0x00 separator mark end of padding, show where hash begins
- **Hash Algo Header**
	- Specify which algorithm is used
	- Adds more constrained bytes in aid of forgery probability, and malleability prevention. 
- $H(m)$ is the actual message hash


#### Signing and verification under PKCSv1.5
- **Signing**
	Compute $H(m)$ 
	Construct padding encoding $EM$ 
	Compute signature over entire message
	Send the signature and the message
- **Verification**
	- Observe hash algo header
	- Compute $s^e$ mod n yielding the padded structure
	- Check all the padding bytes are $0xFF$ 
	- Extract hash algo header
	- Recompute hash over message
	- Check they match

### Problems
- No formal security proof; designed heuristically, its validity is predicated entirely on the difficult of constructing valid encodings under different forgery schemes, particularly selective and existential forgeries, rather than being a computationally hard and verifiable problem
- Idetical messages produce identical signatures under this scheme, determinmsitic.


### RSASSA-PSS
- RSA signature scheme with appendix, address shortcomings of the above
	- With appendix means that $m$ and $s$ are sent separately. 
- Has formal security proof
- Has probabilistic signature scheme that adds a random salt to the process meaning that repeated signatures on the same document will produce different results. 


![](Pasted%20image%2020260329210824.png)
- Includes a salt to the process. Not determinsitic


More cryptographically locked into the signature at 2 points, and is disvcoverably only through one of those paths. To produce a forgery without $d$ one must produce a correctly structured pre-signature message and ensure that the hash of this message is equivalent to the hash of another message, violating collision detection. Must know thee salt, message, and hash of roginaly message. 