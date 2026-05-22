## Block Cipher Mode
- Most messages are not exact multiples of the block size the cipher is expecting
- We need a *mode of operation* to make our SP network primitives more robust and usable, covering arbitrary length messages. 
- Why not use stream cipher
	- Historically, viewed as easier to misuse or implement correctly. 

## Padding
- **Electronic code book (ECB)** and some other modes require plaintext input to be a multiple of the block size to work properly.
- A common padding scheme to ensure messages are a multiple of the block size is **Public Key Cryptography Standards PKCS7**
	1. Padding bytes are always added to the plaintext before it is encrypted(appended)
	2. Each padding byte's value equals the number of padding bytes added.
	3. There is always **at least one padding byte**
	e.g., if 16 bytes of padding, append `0x10` 16 times.
	![](Pasted%20image%2020260301024059.png)
	APPEND NUM OF BYTES NEEDED, WITH THE NUM OF BYTES AS THE VALUE, AT LEAST ONE ALWAYS IN.
	- The reason foor this methodology is that handling padding poorly enables **padding oracle attacks**. We discuss this later
- Note that with PKCS7 even if the plaintext is a multiple of the block size, we still add padding; add entire block of padding bytes. 


## Electronic Codebook(ECB)
 - This is presented as the obvious approach; encrypt each block independently, under the same key $K$
 - **ECB** is deterministic, the same plaintext block yields the same ciphertext block under the same key.
	 ![](Pasted%20image%2020260301024339.png) Visible outlines, because the same plaintext block yields the same ciphertext block under the same key. 
		 AES has no nonce built in unlike ChaCha20. 
- Picture also a bank transfer style message template, where repeated fields across messages that do not change would produce repeated portions in a ciphertext block with the same key, revealing relationships. 
	- CANT BE DOIN THE XXORING OUT THOUGH LIKE WE DID FOR STREAM CIPHERS CAUSE ITS ENTIRELY DIFFERENT. 

### Deterministic vs Probabilistic Encryption
- An encryption scheme is **deterministic** if some plaintext is mapped to a fixed ciphertext if the key remains unchanged; it is a true bijection. 
- Block ciphers and block ciphers in ECB mode are deterministic, but most modern modes of operation are **probabilistic encryption schemes**
- An encryption scheme is said to be probabilistic if it adds randomness to the encryption process to achieve a non-deterministic generation of the ciphertext whilst under the same key $k$
		For example the incorporation of a nonce (whose usage depends entirely on the mode of operation, but remains unencrypted)
	![](Pasted%20image%2020260301024639.png)


### Cipher block chaining
- **CBC** is a block cipher mode where each plaintext block is XOR'd with the previous ciphertext block before encryption
- This uses an **initialisation vector**:
	- This must be unpredictable to an attacker, and unique per encryption under the same key.
	- Often stored/transmitted alongside ciphertext. 
	![](Pasted%20image%2020260301025209.png)
- $y_1 = e_k(x \oplus IV), y_i = e_k(y_{i-1} \oplus x_i)$ 
- Decryption occurs similarly, but XOR occurs after the decryption function like so:
	![](Pasted%20image%2020260522023839.png)
	just decrypt and XOR on either the prev block, or the IV
	![](Pasted%20image%2020260301025347.png)
	This is quite straightofrward to parallelise, as all the information does not require on some part being computed unlike with encryption which required the prior ciphertext to XOR with, we already have the $Y_{i}'s$ here so this is not inherently sequential.
### CBC weaknesses
- Under the same key, the same IV is deterministic. IV + key reuse is catastrophic. 
- Altering ciphertext bits can lead to attacks; flipping blocks in ciphertext block $y_i$ directly flips bits in $x_{i+1}$ for the decryption process due to the bitwise XOR. 
- An encryption algorithm is said to be **malleable** if it is possible to transform a ciphertext into another ciphertext which decrypts to a related plaintext. 
	- For example, suppose bank uses CBC cipher to hide its financial information, and user sends encrypted message saying transfer...
	- If an attacker can guess the message on the wire, and can guess the **format** of the unencrypted message, the attacker could change the amount of the transaction rather reliably by flipping bits in the ciphertext.


### Padding oracles
- An oracle in this context is a theoretical system we can query, that tells us whether, once decrypted, the plaintext has **valid padding**
- System unlikely to tell us directly, but may give away some clue via some side-channel
- Imagine API that receives CBC encrypted auth token
	 ![](Pasted%20image%2020260301030742.png)
	- This leaks the information: did the decrypted plaintext end with valid PKCS7 padding or not
	- Leaks bits because it returns different errors for different failure points. 
	- In CBC, this is enough to recover plaintext. 


### Padding oracle attacks
- In CBC decryption, for some ciphertext block $y_i$:
	- Compute an intermediate block $z_i = d_k(y_k)$
	- Then XOR with the previous block (or IV for the first block) to yield the plaintext $x_i = IV/y_{i-1} \oplus (z_i)$ 
	![](Pasted%20image%2020260301031623.png)
	- An attacker can change the previous block's ciphertext and thereby control the resultant plaintext block in the same positions due to the XOR and malleability
	- So if we can submit chosen ciphertexts and find/determine valid padding, we can actually recover $z_i$ and then recover $x_i$ 


#### Padding oracle attack walkthrough
1. Set the previous block $IV'$ to random, and cycle through all 255 possibilities for the last byte; this is our chosen ciphertext. 
2. We are testing whether the resulting decrypted plaintext block $x_i$ ends with valid padding, for the easiest case, we're hoping to make the last byte of $x_i$ $0x01$ 
	- Note the malleability; attacker cannot see $x_i$; learned whether valid or not from side-channel
	- Obviously, need access to a decryption oracle under this key too, so fairly strong attack model, but don't need more than ciphertext.
3. If the padding is accepted in this easiest case, we know that $IV \oplus z_i = 0x01$ 
	- Once we see that the padding is accepted by altering the previous block that is XORed in, we can rearrange like so below to discover the decrypted variant.
4. Rearrange $IV' \oplus z_i = 0x01$ to $z_i = IV' \oplus 0x01$ 
5. We can repeat this for greater padding lengths, by repeatedly manipulating $IV'$ in order to output a valid padding length and rearranging to discover the corresponding plaintext, this can be repeated across the entire block, and the attacker only needs some of the ciphertext and a padding oracle. 
	This is only an issue because the blocks are not decrypted independently. 



### Counter mode
- Encrypt the once and the counter for num of blocks, and XOR the plaintext in 
	![](Pasted%20image%2020260301193445.png)
	These encryptions are entirely independent, and such avoids the issues that are inherent to ECB such as padding oracle attacks. 

- Pick fixed nonce, for block index $i$ compute a keystream block $s_i = $e_k(n + i)$; behaves exactly like a stream cipher tiwh that XOR
- Decryption obviously is very much the same, compute the same stream and just XOR with the ciphertext $y_i$ to yield the plaintext. 

### Counter benefits
- Avoids repeating ciphertext blocks when plaintext blocks repeat because the counter has changed, resulting in a different keystream for the same block. 
	- ECB does also avoid this because the last ciphertext is used to inform the plaintext; they are not independent encryptions
- CTR can encrypt arbitrary length data without the need for PKCS#7 padding; can XOR partial final blocks like in stream ciphers
	- This is because the resultant stream from the block num and counter is at least as wide as the plaintext block, and the plaintext block does not go into the encrypt function, unlike with say CBC mode, in whih the final block is widened to ensure it is a multiple of the block size. 
	- Removes entire sidechannel/padding oracle vulnerability. 


### Counter issues
- **Nonce reuse** under the same plaintext will result in the same ciphertext being produced, as the keystream will repeat
	- Because it operates like a stream cipher, you can XOR out the keystrrams to yield the XORed plaintexts and then launch a crib dragging attack
- **Malleability**
	- Because $y_i = x_i \oplus s_i$ flipping a bit in the ciphertext flips the same bit in the plaintext, an attacker can predictably modify it, much like with CBC mode (where the XOR is with the prev block)
	- There is no authentication that the message has changed; if can guess content and intercept ciphertext, this could be catastrophic, without necessarily breaking anything
### Galois Counter mode
- GCM, block cipher mode, extends CTR to add authenticity to combat its malleability property
- Very similar to counter mode, adds authentication tag
	- Multiplication in $GF(2^{128})$ with irreducible polynomial $x^128+x^7 + x^2 + x + 1$ 
- Extremely parallelisable as the encryption process itself is entirely separable


#### GCM
![](Pasted%20image%2020260301195850.png)
 - Inject IV as counter 0
 - An increment function + subsequent encryption $e_k$ as it appears in the block copher e.g., AES drives the keystream generation much like in CTR.
 - Counter 1 is the first 'proper key' encrypted and XORed with the first plaintext bytes $x_1$ to yield $y_1$ 
	 - AAd, additional authenticated data, goes into the auth computation and is multiplied by $H$
	 - The result is XORed with $y_1$ then multiplied by $H$ then xored with $y_2$ then... such that it is dependent on **all of the ciphertexts**
	 - This is then XORed with the length of the AAD and the ciphertext
	 - Then finally XOred with the initial IV, depicting counter 0, yielding the tag (we take the most significant bits according to variable $t$), and ensures that the IV has not been tampered
	 - The receiver takes the IV, AAD, the ciphertext, and the lengths, and recomputes the expected tag and compares it to the received tag(which is often appended)