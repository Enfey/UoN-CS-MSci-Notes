## Cryptography
- One part of cryptology, alongside cryptanalysis (analysing crypto-systems, auditing their security, and trying to break them, based on info you have)
	- **Cryptography** = **constructing + analysing mathematical protocols** that prevent third parties from reading plaintext messages
	- **Protocols** = combining algorithms into real systems e.g., TLS, uses multiple cryptographic algorithms to form a secure protocol for secure information exchange. 

### Symmetric cryptography
- Uses, same private key, encrypt + decrypt
	- Secure only if key kept secret
- Very fast, energy efficient, scales well for large amounts of data
	- Just XOR, bit shifts, rotations, easy to compute
- Used for **disk encryption, streaming data, VPNs, databases** (need to be fast)
- Major downside = key management; every communicating pair, needs unique shared secret key. 
	- **num of keys** = $\frac{n(n-1)}{2}$ 
- Implemented via stream + block ciphers.

### Implementation: Stream ciphers
- Use an initial seed key, generate infinite cryptographically-secure pseudo-random keystream, which is then combined with the message (XOR)
	![](Pasted%20image%2020260204181407.png)
- Encryption + decryption, thus same operation
- Good for data, unknown length, often easier, implement, securely.
- Must never reuse same keystream twice (can get XOR combination of two original plaintexts, crib dragging, analysis, work out original message)
	![](Pasted%20image%2020260518152747.png)
### Block ciphers
- Use a key to encrypt a fixed-size block of plaintext, into a fixed-size block of ciphertext
	- Change and permute bits of block, depending on key. 
- To encrypt a long message:
	- Split into blocks, add padding, use mode of operation.
		- GCM(AEAD), CTR, XTS(disk encryption)
		- Use mode of operation, as if we did blocks independently, identical plaintext blocks yield identical ciphertext blocks, e.g., CTR, counter for each block, encrypt counter, XOR result with plaintext. 
	- More integrity via modes of operation (AEAD), stream ciphers, can just flip bits in ciphertext, affect plaintext.
- Key split into rounds + mode of operations. 

### SP-networks
- Block cipher construction method; based on 3 ideas
- **Substitution**
	- introduce non-linearity, replace small chunk of bits with other chunks, via lookup table, invertible, fixed for algorithm.
- **Permutation**
	- spread non-linearity across block (global) by rearranging bits, changing positions. otherwise each chunk isolated, could target easily
- **Key-mixing**
	- tied everything to the secret key
	- usually derive multiple keys from master key, and apply in multiple rounds with subsitution and permutation
- **GOAL** = make relationship between plaintext, key, ciphertext so complex, appears random
	- 2 properties needed:
		1. **Confusion**  - obscure relationship between key and ciphertext
		2. **Diffusion** - spread influence of plaintext bits, across many ciphertext bits. 
- **Key-mixing**
	- Take master key, derive round keys via **key expansion**, ensure each SP round behaves differently.
		![](Pasted%20image%2020260518154342.png)
- Sboxes/substitution tables are public, same with actual structure, only secret = key. 
	- Repeated rounds+permutation intensity control avalanche characteristics(tiny change in input, results in drastic, random change in output)

### Symmetric algorithms
- DES
- 3DES
- AES
- ChaCha20
- TLS 1.3 - permits only latter 2

## Cryptographic attack models
- Attack model, describe capabilities + knowledge attacker assumed to have when breaking a system. 
	- Used to evaluate protocol/algorithm strength. 
- A cryptosystem, only a strong, as the attack model it can withstand
- Weakest to strongest:
	- **Ciphertext-Only**
		- Only know ciphertext, goal = recover plaintext or key
	- **Known-Plaintext**
		- Knows some **plaintext-ciphertext pairs**, goal = use known pairs to deduce key/decrypt ciphertexts
	- **Chosen-Plaintext**
		- Can choose plaintexts, obtain their ciphertexts, do not have the key. Detect patterns in encryption, infer structure of algorithm.
		- Learn info about the key
	- **Chosen-Ciphertext**
		- Can choose ciphertexts and obtain plaintexts (cannot ask for decryption of exact target ciphertext)
		- Can modify ciphertext, extract info, gradually decrypt
		- Goal is to recover key or decrypt message.
	- **Related-key attack**
		- Study encryptions made with diff keys that are mathematically related, observes ciphertexts under multiple keys, knows how they are related
		- If key schedule is weak, key changes may cause predictable changes in ciphertext.


### Key sizes
- Symmetric encryption, **if secure**, is only susceptible to brute force
- 128 bit key, $2^{128}$  to brute force. 

### Stream vs Block Ciphers
- **Stream**
	- Suited continuous streams, unknown length
	- No padding
	- Easier to implement
	- Fewer side channel risks
- **Block**
	- More versatile, can be used, build other cryptographic primitives,, modes give integrity etc.
	- Benefit from hardware acceleration
	- Larger ecosystems

## Asymmetric cryptography
- **Uses two keys**
	- Public key, private key
- Used for **key exchange** and **signatures**
	- Encrypt with public key, decrypt with private. 
	- Sign with private, verify with public
- Much more expensive than symmetric
- Key management easier, each user in network has one private, one public key, thus 2n keys, rather than unique key for each pair, scales better. 
- Asymmetric/public key cryptography, hinges upon the premise, it is infeasible to calculate a private key from a public key
	- Achieved, via NP hard problems
		- RSA integer factorisation
		- DH DLP


### Diffie-Hellman Key Exchange
- Method, generating symmetric cryptography key (or at least, a shared secret from which to derive a key)
- Permit two parties, agree, shared secret, over insecure channel, exchange public values, secret values never transmitted, both sides compute same shared key. 
	- Both parties agree on $p$ and generator $g$ of a multiplicative group $p$.
	- Each party chooses private exponent, raising $g$ to it yielding public values:
		- $A = g^a \ mod \ p$ 
		- $B = g^b \ mod \ p$
	- These public values are sent over the network, and the shared secret is computed
	- Each side combines their private exponent with the other party’s public value
			- $S = B^a \ mod \ p = (g^b)^a = g^{ab} \ mod \ p$ 
			- $S = A^b \ mod \ p = (g^a)^b = g^{ab} \ mod \ p$ 
		- Attacker cannot break without solving DLP (discrete logarithm problem).

#### MITM DHKE
- An opponent Carol intercepts Alice's public value $A$ and sends her own public value to Bob  $M_1$ 
- When Bob transmits $B$, Carol substitutes it with her own $M_2$ pretending to be Bob. 
- Carol and Alice agree on one shared secret, and Carol and Bob agree on another. 
- There is no proof that $B$ or $A$ came from Bob or Alice.
- Carol can now decrypt, read, modify, and re-encrypt all conversations between the two parties. 
- This warrants digital signatures.

### Digital Signatures
- Mathematical scheme, verifying **authenticity** of digital messages.
- Valid digital signature, gives recipient confidence, message came from sender, known to the recipient
- Provides
	- **Authenticity**
	- **Integrity**
	- **Non-repudiation**
- Digital signature scheme composed of three algorithms:
	- **1. Key generation algorithm**
		- Select private key at random from set of possible private keys, output private key and public key. 
	- **2. Signing algorithm**
		- Given message + private key, computes signature.
	- **3. Signature verifying algorithm**
		- Given message and signature and public key, either accepts or rejects the messages claim to authenticity.
- DHKE, server sign DH parameters, client verifies, using known public key.

### Certificates and Public Key Infrastructure
- Public keys, meaningless, without **identity binding**
	- Prevent MITM attacks; where someone else sends u a public key impersonating the server, and you send them info. 
- A **certificate** binds a public key to an **identity**
	- Digitally signed by CA, third party entity responsible for certifying ownership of public keys
- Certs contain:
	- Public key
	- Validity period
	- Signature from CA
	- Server identity
- Systems trust small set of root CAs; a compromised CA is disastrous, all certificates from them cannot be trusted. 


## Post-Quantum cryptography
- Quantum computers, threat, some cryptography, quantum-hardened algorithms and protocls not adopted in widespread scale
- Shors algorithm breaks asymmetric cryptography
- Grovers algorhthm weakens symmetric, but can be mitigated by just doubling the key sizes
- Real threat = harvest now decrypt later, bad for long lived secrets
- Post-quantum, much larger keys, computation, but less field tested, current strategy is hybrid. 