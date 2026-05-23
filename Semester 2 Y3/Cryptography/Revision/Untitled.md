## MAC
- Message authentication code is a cryptographic tag computed over a message $m$ to prove the message was not modified, and came from someone who knows a shared secret
- Use symmetric keys; sender and receiver share the same secret
	![](Pasted%20image%2020260330213546.png)
	Where $h(k | m) = MAC$, appended to the end of the message, is a hash over the prepended $k$ and $m$.
	Proves know the key, and the message contents
- Not provide non-repudiation, nor confidentiality
- Receiver recomputes mac using their shared secret and hashing algorithm over the message.

### Length extension attacks
- Merkle damgard iterates using the previous hash value $H_{i-1}$ 
- Can resume hashing by initialising the compression function to be the previous hash
- Can hash furtherdata by taking an existing hash and continuing the hashing function, which is worrying. 
- Early systems used $MAC = SHA256 (k || message)$; problematic because while $k$ is not known, the most recent hash value is. 
	1. Observe m and the hash over k and m
	2. Create a new attack message
	3. Initialise the hash function with the prior hash value
	4. Compute the hash over the attack message. 


- The attacker must estimate the padding though.
	- The reason is the attacker does not know the key length; the padding used depends on the total length of the entire message of the key


### HMAC
- SHA-256 is a merkle-damgard construction that reveals internal state after hashing
- An ttacker can continue hashing and forge a valid hash for a message of their choice without knowing the key
- As an alternative, hash message authentication code $HMAC$ can be used in place of a standard MAC
	- The problem we solve is that the hash cannot be resumed. 
		$$HMAC(k, m) = H ((k \oplus opad) || H(k \oplus ipad)||m))$$
- HMAC prevents length extension attacks; the inner hash is never seen and the result of the outer hash is invertible, so it is infeasible to perform this attack
	- Even if they could length-extend an existing message.. coildnt lol.


### AEAD
- The pattern of encrypting data and producing a tag over it and sending both the ciphertext and the tag became so common that modern ciphers integate both operations
- AEAD provides confidentiality via encryption, and integrity+authenticity via the MAC
	- The MAC is no longer computed manually; the cipher mode does it alongside
- AES-GCM is a cipher mode over AES construction that outputs a ciphertext and an authentication tag, over which the ttag is computed with rgeard to some additional data, length of addityional data and ciphertext, and the ciphertext itself
	- So tampering with either the ciphertext or initialisation, causes tag verification on the reciever's end to fail.
- - **ChaCha20-Poly1305**
	- Uses ChaCha20 for data encryption and Poly1305 as a one-time MAC.
- The reason for AEAD cippher scheems is because manual constructions were often implemented incorrectly, led to padding oracle attacks becoming feasible, etc. 