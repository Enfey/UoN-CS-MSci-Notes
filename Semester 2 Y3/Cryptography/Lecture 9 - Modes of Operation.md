## Block Cipher modes
- Most messages aren't usually exact multiples of the block size and are often long streams of data.
- We need a *mode of operation* too apply a block cipher repeatedly to cover arbitrary-length messages
- *Why not use a stream cipher?*
	- Stream ciphers, historically, were viewed as easier to misuse or implement correctly, those modern stream ciphers like ChaCha20 are widely trusted and deployed. 

## Padding
- Electronic code book (ECB) and some other modes require plaintext length to be a multiple of the block size
- A common padding scheme to ensure messages reach this size is **Public Key Cryptography Standards PKCS7**:
	1. Padding bytes are always added to the plaintext before it is encrypted (appended).
	2. Each padding byte's value equals the number of padding bytes added. 
	3. There is always at least one padding byte
	![](Pasted%20image%2020260301024059.png)
	- E.g., if needed 16 bytes of padding, append `0x10` 16 times. 
	- The reason for this methodology is that bad padding handling becomes a security vulnerability, which can see later on with **padding oracle attacks**. 
- Note that with PKCS7 even if the plaintext length is already an exact multiple of the block size, we still add padding; we would actually add a while extra block of padding bytes. 

## Electronic Codebook
- This is presented as the obvious approach: encrypt each block independently with the same key. 
- ECB is deterministic: same plaintext block + same key = same ciphertext block. 
	![](Pasted%20image%2020260301024339.png)
	Visible outlines because identical pixel blocks map to identical ciphertext blocks. 
- Picture also a bank transfer style message template, where repeated fields across messages that do not change would produce repeated portions in a ciphertext block with the same key, revealing relationships. 


## Deterministic vs Probabilistic Encryption
- An encryption scheme is **deterministic** if some plaintext is mapped to a fixed ciphertext if the key remains unchanged. 
- Block ciphers and block ciphers in ECB mode are deterministic, but most modern modes of operation are **probabilistic**.
- An encryption scheme is **probabilistic** if it adds randomness to the encryption process to achieve a non-deterministic generation of the ciphertext e.g., unique nonce (must not repeat under the same key).
	![](Pasted%20image%2020260301024639.png)

### Cipher Block Chaining
- CBC (Cipher Block Chaining) is a block cipher mode where each plaintext block is XOR'd with the previous ciphertext block before encryption
- The first block uses an initialisation vector
	- This must be unpredictable to an attacker and must be unique per encryption under the same key. 
	- Often stored/transmitted alongside the ciphertext (often prefixed).
	![](Pasted%20image%2020260301025209.png)
- Decryption occurs similarly, but $XOR$ occurs after the decryption function to yield $x_i$ and then inverse with the IV, and $y_{i-1}$ respectively.
	![](Pasted%20image%2020260301025347.png)
	This is also quite straightforward to parallelise; all the information does not require on some prior part being computed unlike with encryption which required the prior ciphertext to $XOR$ with; only need the $y's$ here so this is not inherently sequential. 

#### CBC weaknesses
- If the **IV** remains unchanged within that key cycle, relationships between messages can be leaked, especially if they contain similar content, as this reduces the probabilistic effect an IV has. Under the same key, the same IV is deterministic. 
- Altering ciphertext bits during the decryption process can lead to attacks; flipping bits in ciphertext block $y_i$ directly flips bits in $x_{i+1}$ due to the $XOR$. 
	![](Pasted%20image%2020260301025834.png)
- An encryption algorithm is said to be **malleable** if it is possible to transform a ciphertext into another ciphertext which decrypts to a related plaintext. 
	- For example, suppose that a bank uses a CBC block cipher to hide its financial information, and a user sends an encrypted message containing, say, `"TRANSFER $0000100.00 TO ACCOUNT #199."` 
	- If an attacker can modify the message on the wire, and can guess the format of the unencrypted message, the attacker could change the amount of the transaction, or the recipient of the funds, e.g. `"TRANSFER $0100000.00 TO ACCOUNT #227"`. 
	- Malleability does not refer to the attacker's ability to read the encrypted message. Both before and after tampering, the attacker cannot read the encrypted message.

### Padding Oracles
- An oracle in this context is a theoretical property/system we can query that tells us whether, once decrypted, the plaintext has **valid padding**.
- A system is unlikely to tell us directly, but it may give away some clue. 
- Imagine an example API that receives a CBC encrypted authorisation token. 
	![](Pasted%20image%2020260301030742.png)
	- This leaks the information: did the decrypted plaintext end with valid PKCS#7 padding, or not?
	- Leaks bits because it returns different errors for different failure points.
	- In CBC, this is enough to recover plaintext. 

### Padding Oracle Attacks
- In CBC decryption, for some ciphertext block $y_i$:
	- Compute an intermediate block $z_i = d_k(y_i)$ 
	- Then $XOR$ with the previous ciphertext block (or IV for the first block) to yield the plaintext: $x_i = z_i \oplus (IV \ or  \ y_{i-1})$
	![](Pasted%20image%2020260301031623.png)
- An attacker can change the previous block's ciphertext (or IV) and thereby control the resulting plaintext block $x_i$ without knowing the key, because $XOR$ happens after $d_k(y_i)$.
- So if we can submit chosen ciphertexts and find/determine valid padding, we can actually recover $z_i$ and then recover $x_i$. 

#### Padding Oracle Attack Walkthrough
1. Set the previous block $IV'$ to random, and cycle through all 255 possibilities for the last byte, this is our chosen plaintext. 
2. We are testing whether the resulting decrypted plaintext block $x_i$ ends with valid padding, for the easiest case, we're hoping to make the last byte of $x_i$ equal to $0x01$.
	- Note the malleability property: the attacker cannot see $x_i$ - only learn via a yes/no signal e.g., API response from the server. 
	- We assume access to this decryption endpoint has already been acquired. 
3. If the padding is accepted in this easiest case, we know $IV' \oplus z_i = 0x01$ 
4. Rearrange $IV' \oplus z_i = 0x01$, $z_i = IV' \oplus 0x01$ 
5. We repeat this for greater padding lengths; once we know $z_i[16]$ we can manipulate the IV to output a plaintext with a padding length of 2 with values 2, and rearrange to acquire $z_i[15]$, and so on and so forth, eventually recovering $z_i$, at which point we can recover the original plaintext through $XOR$ with the previous block.


### Counter Mode
 - Encrypt a nonce + a counter and use this to mask the plaintext with $XOR$ 
 - The encryption is applied to $XOR$ only.
 - This is very easily parallelised as it is not reliant on prior output; the key and the nonce are fixed under the encryption cycle, just increment the counter for each block, yielding $y_i$'s
	 ![](Pasted%20image%2020260301193445.png)
- CTR turns a block cipher into a keystream generator:
	- Pick a fixed Nonce
	- For each block index $i$, compute a keystream block $s_i = e_k(Nonce + i)$ 
	- Mask the plaintext with $XOR$, $y_i = x_i \oplus s_i$ 
- This behaves exactly like a stream cipher; generate pseudorandom stream and $XOR$ with plaintext. 
- Decryption is the same operation as XOR cancels, so we compute the same $s_i$ and $XOR$ with the ciphertext, $y_i$. 

#### Counter benefits
- CTR avoids repeating ciphertext blocks when plaintext blocks repeat  because the counter has changed; the keystream blocks differ for a different block. 
- CTR can encrypt arbitrary length data without PKCS#7 padding; can XOR partial final blocks much like in stream ciphers, which removes the entire class of padding sidechannel/oracle vulnerabilities that CBC + PKCS#7 have. 
- Parallelism. 

#### Counter issues
- Nonce reuse
	- If the same key $k$ is used with the same Nonce, then the same keystream repeats
		![](Pasted%20image%2020260301194503.png)
	- $y_i = x_i \oplus s_i$, $y'_i = x'_i \oplus s_i$ 
	- If we XOR the ciphertexts, we get $y_i \oplus y'_i = x_i \oplus x'_i$ 
		- From here, could launch a crib dragging attack; relationship between plaintexts leaked, often able to recover parts of, if not the whole plaintext. 
- Malleability (and why GCM is needed)
	- Because $y_i = x_i \oplus s_i$, flipping a bit in the ciphertext flips the same bit in the plaintext. 
	- $y_i \oplus \Delta \implies x_i \oplus \Delta$ 
	- An attacker can predictably modify plaintext, and with this mode there is no authentication that the message has changed. If the attacker can guess contents and intercept the ciphertext, this could be catastrophic whilst not being a 'direct' attack. 

### Galois Counter mode
- Galois Counter Mode (GCM) is a block cipher mode which extends CTR to add authenticity to combat its malleability property
- Very similar to counter mode, but adds an authentication tag.
	- Multiplication in $GF(2^{128})$ with irreducible polynomial $x^{128} + x^7 + x^2 + x + 1$ 
- Extremely parallelisable which will become obvious when viewing its construction below, and is robust against message alteration. 

#### GCM diagram and explanation
![](Pasted%20image%2020260301195850.png)
- Inject an IV as counter 0; our nonce. 
- An increment function $incr$ + subsequent encryption $e_k$ as it appears in the block cipher e.g., **AES** drives the keystream generation much like in CTR.
- Counter 1 is the first 'proper key' encrypted and $XOR$'d with the plaintext to yield $y_1$
	- AAD, additional authenticated data, goes into the authentication computation and is multiplied by $H$
	- The result is $XOR'd$ with $y_1$
	- That result is then multiplied by $H$, etc etc. 
		- $H = e_k(0^{128})$ , forged from key.
		- ![](Pasted%20image%2020260301203803.png)
		- H will differ, generate a polynomial at each ciphertext block and reduce it, such that the final $S_m$ will be different if the ciphertext has changed; position sensitive. 
		- Expands to $S = (S \oplus y_i) \cdot H$ per block. 
- A special block is appended, $len(A) ||len(C)$, which contains the lengths of AAD and the ciphertext. 
- The result of encrypting the initial IV, depicting Counter 0, is $XOR'd$ with the $H$ result of the final synthetic block, yielding the tag (we take the most significant bits via $MSB_t$, taking the $t$ most significant bits).
- Note that the tag generation is logically separate from the encryption, but uses its results and the IV. 
- The receiver takes the IV, ciphertext $C$, AAD A, and lengths len(A), len(C) and recomputes the expected tag and comapres it to the received tag
	- Tag is often appended or stored as a separate field. 
	- So if the attacker has altered the ciphertext, we will know because the threat model we defend against is that the attacker can modify what is sent over the wire; compute $T'$ at the other end, will see that $T' \neq T$ and reject.

