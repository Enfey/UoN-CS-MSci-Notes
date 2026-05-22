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
	- XOR with the 


#### Padding oracle attack walkthrough
