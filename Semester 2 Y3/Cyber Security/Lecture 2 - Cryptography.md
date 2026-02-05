# Cryptography
- Cryptography is one part of **cryptology**, alongside **cryptanalysis** (breaching cryptographic security systems and gaining access to contents, based on information attacker has).
	- **Cryptography** = constructing + analysing protocols that prevent third parties from reading private messages.
	- **Protocols** - combining algorithms into real systems e.g., TLS.

## Symmetric Cryptography:
- Uses the same **key** to **encrypt** and **decrypt**
	- Anyone with the key can *lock and unlock*, secure only if the key is kept secret
- Very fast, very energy efficient, and scales well for large amounts of data. 
	- Computationally cheap in comparison to asymmetric cryptographical methods e.g., XOR, bit shifts + rotations, modular arithmetic, very straightforward to compute.
- Used for(things that need to be fast):
	- Disk encryption
	- Streaming data
	- Databases
	- VPNs
- The major downside is key management; every communicating pair needs a unique shared secret key.
	- **num of keys** = $\frac{n(n-1)}{2}$ 
- Implemented via **stream ciphers** and **block ciphers**
### Implementation: Stream Ciphers
- Stream ciphers use an initial seed key to generate an infinite pseudo-random keystream, which is then combined with the message (typically using XOR, as this is reversible if applied twice.)
	![](Pasted%20image%2020260204181407.png)
- Encryption and decryption, thus the same operation
- Good for data of unknown length, and often easier to implement securely. 
- Must **never** reuse the same keystream twice [Lecture 2 - Cryptography](Lecture%202%20-%20Cryptography.md)
### Implementation: Block Ciphers
- Block ciphers use a key to encrypt a **fixed-size block** of plaintext into a **fixed-size block** of ciphertext
	- Change and permute bits of the block depending on the key
- To encrypt long messages:
	- Split into blocks, add padding, use a mode of operation
		- Mode of operations: CTR(counter), XTS(disk encryption), GCM(authenticated encryption)
		- Use mode of operation, as if we encrypted each block independently, identical plaintext blocks would yield identical ciphertext blocks, CTR for example, counter for each block, encrypt the counter, then XOR the result with the plaintext.
	- More opportunity for integrity with block ciphers via modes of operation, stream ciphers do not have any, attackers can flip bits in ciphertext and predictably affect plaintext.
- Technically doesnt use same key all the time, depends on **rounds** and mode of operation. 

#### SP-Networks
- **Substitution-permutation network**, way of constructing **block cipher** by repeatedly applying three ideas:
	- **Substitution**: introduce non-linearity (local)
		- Replace small chunks of bits with other chunks using a lookup table, invertible, fixed for the algorithm.
	- **Permutation**: spready that non-linearity across the block (global)
		- Rearrange bits, only changes positions, if only substituted, each chunk would stay isolated, could target small independent parts, want to spread influence.
	- **Key mixing**: tie everything to the secret key
	- Applied in multiple rounds
- Goal is to make relationship between plaintext, ciphertext, and key so complex that the cipher looks utterly random to an attacker. 
	- 2 properties needed for secure ciphers:
		1. **Confusion** - obscure relationship between key and ciphertext
		2. **Diffusion** - spread the influence of each plaintext bit across many ciphertext bits
- **Key mixing**
	- Take the master key, and derive round keys via key expansion, to ensure each round of substitution and permutation on the ciphertext behaves differently.
		![](Pasted%20image%2020260204184044.png)
- The S-boxes/substitution tables are public, and so are the permutation/diffusion rules, the only thing kept secret in SP-networks is the key. 
- Repeated rounds+permutation intensity control avalanche characteristics.


### Symmetric algorithms
- DES - broken due to short key
- 3DES - sometimes used in legacy systems, slow
- AES - modern dominant block cipher
- ChaCha20 - dominant stream cipher
- TLS 1.3 only permits the latter two.  
	![](Pasted%20image%2020260204184502.png)

### Cryptographic attack models
- Attack models describe the capabilities and knowledge an attacker is assumed to have when trying to break a cryptographic system.
- Used to evaluate how strong an algorithm or protocol really is. 
- ***A cryptosystem is only as strong as the attack model it can withstand.***
- From weakest attack strength to strongest:
	- **Ciphertext-only**
		- Only know the ciphertext, goal is recover the plaintext or the key.
	- **Known-plaintext**
		- Knows some plaintext-ciphertext pairs, goal is to use known pairs to deduce the key or decrypt other messages.
	- **Chosen-plaintext**
		- Can choose plaintexts and obtain their ciphertexts, do not have the key, , detect patterns in encryption and infer structure of algorithm.
		- Goal is to learn info about the key.
	- **Chosen-ciphertext**
		- Can choose ciphertexts and obtain their decrypted plaintexts. Can see decrypted output, but cannot ask for decryption of exact target ciphertext.
		- Can modify ciphertext slightly, and extract info, gradually decrypt messages. 
		- Goal is to recover the key or decrypt a protected message.
		- Strongest realistic standard.
	- **Related key-attack**
		- Instead of attacking single unknown key, attacker studies encryptions made with different keys that are mathematically related
		- Attacker observes ciphertexts under multiple keys
		- Knows how the keys are related
		- If algorithm key schedule is weak, key changes may cause predictable changes in ciphertext.
		- Mostly a theoretical concern. 
#### Key sizes
- Symmetric encryption, if secure, is only susceptible to **brute force**
- 128 bits key means $2^{128}$ required to brute-force
- Very far from practical threat for high bit keys e.g., 512
#### Stream vs Block ciphers
- Stream ciphers
	- Naturally suited for continuous streams, possibly of unknown length
	- No padding required
	- Easier to implement correctly
	- Fewer side-channel attack risks. 
- Block ciphers
	- More versatile, can build other useful cryptographic primitives
	- Benefit from hardware acceleration
	- Larger ecosystem today e.g., web, disk encryption. 


## Asymmetric cryptography
- Uses **two keys**
	- Public key(shared)
	- Private key (secret)
- The analogy we can use is that of a mailbox, anyone can drop mail in, but only the owner can open it. 
- Used for **key exchange** and **signatures**
	- Encrypt with public key, decrypt with private 
	- Sign with private key, verify with public key
- Much more expensive than symmetric
- Key management much easier: each user in a network has one private key and one public key, thus n keys (key pairs), rather than a unique secret key for each communicating pair. 
- Asymmetric/public key cruptography hinges upon the premise that it is **computationally infeasible** to calculate a private from a public key
- In practice, this is achieved through reduction to intractable/computationally hard mathematical problems (NP)
	- RSA integer factorisation
	- Diffie-Hellman discrete logarithms

### Key Exchange - Diffie-Hellman
- Diffie-Hellman key exchange is a method of generating a symmetric cryptography key over a public channel
	![](Pasted%20image%2020260205181750.png)
- Permits two parties to mathematically agree a shared secret over an insecure channel, public values are exchanged, and secret values are never transmitted, and both sides compute the same shared key.
	- Both parties agree on a large prime $p$, and a generator $g$ of a multiplicative group modulo $p$. These are public, and often standardised. 
	- Each party chooses a random private exponent/key($a$ and $b$), must be large and random, and never reused long-term
	- Each side computes a public value
		- $A = g^a \ mod \ p$ 
		- $B = g^b \ mod \ p$ 
	- Then these public values/keys are sent over the network, and the shared secret is computed
		- Each side combines their private exponent with the other party’s public value
			- $S = B^a \ mod \ p = (g^b)^a = g^{ab} \ mod \ p$ 
			- $S = A^b \ mod \ p = (g^a)^b = g^{ab} \ mod \ p$ 
		- Attacker cannot compute $g^{ab}$ without solving DLP (discrete logarithm problem).
- Vulnerable to man in the middle attacks:
	- An opponent Carol intercepts Alice public value $A$ and sends her own public value to Bob $M_1$. When Bob transmits $B$, Carol substitutes it with her own $M_2$ and sends it to Alice, pretending to be Bob. Thus, Carol and Alice agree on one shared key, and Carol and Bob agree on another shared key as the exchange has been manipulated. There is no proof that $B$ or $A$ came from Bob or Alice.  Carol could decrypt, read, modify, and re-encrypt all conversations between the two parties. This warrants digital signatures and certificates.

### Digital signatures
- Mathematical scheme for verifying the authenticity of digital messages or documents. 
- Valid digital signature on a message gives a recipient confidence that the message came from a sender known to the recipient. 
- Provides:
	- **Authenticity**
	- **Integrity(non modified)**
	- **Non-repudiation**
- A digital signature scheme consists of three algorithms:
	1. A **key generation algorithm**
		Selects a **private key** at random from a set of possible **private keys**, algorithm outputs **private key** and corresponding **public key**.
	2. A **signing algorithm** that given a **message** and a **private key**, produces a **signature.**
	3. A **signature verifying** algorithm, that, given the **message**, **public key,** and **signature,** either **accepts** or **rejects** the messages claim to authenticity.
- For DHKE, server can sign DH parameters, client verifies signature using known public key. 

### Certificates and Public Key Infrastructure
- Public keys are meaningless with **identity binding**
- A **certificate** binds a public key to an identity
	- These are digitally signed by a certificate authorities, a third-party entity responsible for certifying the ownership of a public key by the named subject of the certificate to facilitate trust in that public key.
- Certificates contain:
	- Subject identity
	- Public key
	- Validity period
	- Signature from CA to verify. 
- System trusts a small set of root CAs, but this centralisation is operationally fragile, and compromised CAs are disastrous, as it means all their certificates cannot be trusted.
### Public Key Algorithms ![](Pasted%20image%2020260205184753.png)

### Post-Quantum Cryptography
- Quantum computers are a real-threat to some cryptography, quantum hardened algorithms do exist but are not adopted on widespread scale.
- Shors algorithm breaks asymmetric cryptography
- Grovers algorithm weakens symmetric cryptography, but this can be mitigated by just doubling the key sizes.
- The real threat, is that encrypted traffic can be recorded en masse today, and decrypted years later once quantum exxists, which is bad for long-lives secrets.
- Post quantum cryptography has much larger keys, much larger keys, more computation, and is less field tested, thus the current strategy, is a transition period where hybrid cryptography is used, both classical+post quantum.

### TLS
- Protocol that combines many cryptographic primitives and carefully orchestrates them
- Client already has **trust store** of root CA public keys
- Server already has certificate (issued by CA), and the private key corresponding to the certificates public key.
	1. Client sends supported TLS versions, ciphers, random nonce, and ephmeral DH public key
	2. Server sends chosen cipher suite, random nonce, Servers ephemeral DH key
	3. Each side computes $S = g^{ab}$, still vulnerable to MITM attacks
	4. Server sends certificate, and signature over the handshake. 
	5. Client verifies certificate using trusted CA public keys, knows was issued by the CA, and then verifies the servers signature using the public key contained in the certificate.
	6. Both sides use the DH secret and derive symmetric session keys
	7. Often send MACs over the handshake (called finished messages)
	8. All traffic now uses symmetric encryption