 
## Message Authentication Codes
- A **MAC** is a cryptographic tag over a message $m$ that proves the message was not modified (integrity) and proves the message came from someone who knows a shared secret (**authentication**)
- Unlike digital signatures, MACs use **symmetric keys** where the sender and receiver share the same secret.
	![](Pasted%20image%2020260330213546.png)
	Where $h(k | m) = MAC$, appended to the end of the message, is a hash over the prepended $k$ and $m$.
- They do not provide non-repudiation(symmetric), nor confidentiality for they do not encrypt the message.
- The receiver recomputes the MAC using their shared secret and hashing algorithm over the message, and if it matches the appended MAC, then the message is authentic and hasn't been tampered with.
## Merkle-Damgard Reminder
![](Pasted%20image%2020260330213818.png)
- The Merkle-Damgard iterates using the previous hash value $H_{i-1}$ 
- This means that you can resume hashing by initialising the incoming state to the compression function to be the previous hash. 
- This means that hashes of secret data could be used to produce hashes of $\text{secret data | more data}$ 

### Length-Extension Attacks
- Early systems used $MAC = SHA256 (k || message)$ 
- This is problematic; the attacker doesn't know $k$, but the most recent hash value $h(k|m)$ is known.
- Steps of the attack:
	1. Observe $m$ and $h(k|m)$ 
	2. Create a new attack message $m | x$ 
	3. Initialise hash function with an internal state of $h(k | m)$ 
	4. Complete the hash with $x(s) to produce h(k | m | x)$ 

#### Example Length Extension Attack
- Suppose a bank transfer sends a message, authenticated using a MAC:
	![](Pasted%20image%2020260330214727.png)
- This message is appended to a secret key to produce $k|m$:
	![](Pasted%20image%2020260330214746.png)
- The data to be hashed is padded such that it fits the format of SHA's blocks (observed in the last lecture):
	![](Pasted%20image%2020260330214830.png)
- A final hash/MAC $h(k|m)$ is produced:
	![](Pasted%20image%2020260330214852.png)
- The attacker estimates what the padding must be
	- The reason for this is that the attacker does not know the key length; the padding depends on the total length of the entire message and the key, but if the key is unknown then we cannot determine the padding that appears in the final block. 
	- This is problematic for reasons explained below:
		- The attacker observes $H = SHA256(k || m)$ 
		- Internally, SHA256 has processed $k || m || PAD(k||m)$ with $H$representing the last state after the padding block has already been processed.
		- When the attacker performs the length-extension attack, they produce $H' = SHA256(k||m||PAD||x)$ 
		- They need the server to reproduce this value; when the server recomputes the MAC, it will not yield this as the padding will no longer appear in the same place as the message length and contents have changed.
		- Whatever the attacker sends must be a message such that $SHA256(k || m) = H'$
		- The only way the server can reach $H'$ is if it processes $k || m || PAD || x$ so it has the same padding bytes in the same position that they attacker did.
		- Therefore the attacker must send $m || PAD || x$ so that the server hashes the same byte stream that the attacker simulated so that the MAC is approved, otherwise it would just do $m || x$ and fail.
- After estimating the padding, a new extended message is crafted which appends the padding and the message $x$.
	![](Pasted%20image%2020260330221938.png)
- We bootstrap a SHA-256 instance with the prior hash $k || m || PAD$ and introduce our message blocks:
	![](Pasted%20image%2020260330222055.png)
- This yields a final hash that we can replace in the packet alongside our crafted message.

## HMAC
- A naive MAC fails because SHA-256 is a Merkle-Damgard construction that reveals the internal state after hashing
- As described above an attacker can continue hashing from that state and forge a valid message without knowing the key.
- As an alternative, hash message authentication code *HMAC* can be used in place of a standard MAC.
	- The problem we solve is that the hash cannot be resumed, which is a fundamental weakness of the Merkle-Damgard construction. 
$$HMAC(k, m) = H ((k \oplus opad) || H(k \oplus ipad)||m))$$
- We take the key $k$ and duplicate it, and $XOR$ it with some inner padding and outer padding, respectively.
	![](Pasted%20image%2020260330231026.png)
- We perform the first inner hash with $i \ key \ pad$ and the appended message:
	![](Pasted%20image%2020260330231116.png)
	- On its own, this is technically length extendable, we could then resume here and calculate more hash, so we hash again.
- We append $hash \ 1$ to the $o \ key \ pad$ and then take the hash of that to compute the HMAC.
	![](Pasted%20image%2020260330231238.png)
- HMAC completely prevents length extension attacks. Even if the attacker extends the message hash, $hash \ 1$ they cannot compute the necessary outer hash result to fool the server because they lack the secret key. 
	- The inner hash is also never seen, and the result of the outer hash is invertible, so it is entirely infeasible to perform this attack.
## Authenticated Encryption with Associated Data (AEAD)
- The pattern of encrypting data $Enc_k(m)$ and producing a tag over it $MAC_k(ciphertext)$ and sending the ciphertext and tag became so common that modern ciphers integrate both operations. 
- **AEAD** provides **confidentiality** via encryption, and integrity + authenticity via the MAC.
	- So the MAC is no longer computed manually; the cipher mode does it internally.
- By associated data, we mean data that is authenticated by the MAC but is not encrypted.
	- For example protocol headers, packet numbers, primary key for database
- **AES-GCM**
	- Outputs ciphertext and an authentication tag, of which the tag is computed over associated data, ciphertext, and lengths
	- So tampering with either ciphertext or associated data causes tag verification to fail.
- **ChaCha20-Poly1305**
	- Uses ChaCha20 for data encryption and Poly1305 as a one-time MAC.
- The reason for AEAD cipher schemes is because manual constructions were often implemented incorrectly which often lead to padding oracle attacks becoming feasible, MAC bypassing etc.
	- AEAD modes enforce a provably secure composition of encryption and authentication such that they do not have to worry about the implementation. 


## TLS
- Transport Layer Security (TLS) is a Protocol that provides two security properties for network sessions:
	- **Authentication**
	- **Confidentiality(encryption)**
- It further provides **integrity via use of AEAD ciphers**.
- It provides these properties over two layers:
	1. **Handshake layer**
		- Used to establish session keys as well as authenticate either party - typically authenticate the server via a public-key certificate provided by an intermediate CA.
	2. **Record layer**
		- Uses the keys established during handshake layer amongst other information to encrypt application-level packets.

### Handshake Overview
- 

## PKI