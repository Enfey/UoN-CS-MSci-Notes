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