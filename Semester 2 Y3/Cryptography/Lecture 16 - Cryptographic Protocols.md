 
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
	4. Complete the hash with $x(s)$ to produce $h(k | m | x)$ 

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
		- Internally, SHA256 has processed $k || m || PAD(k||m)$ with $H$ representing the last state after the padding block has already been processed.
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
- It further provides **integrity via use of AEAD ciphers** (in TLS 1.3, all non AEAD ciphers have been removed).
- It provides these properties over two layers:
	1. **Handshake layer**
		- Used to establish session keys as well as authenticate either party - typically authenticate the server via a public-key certificate provided by an intermediate CA.
	2. **Record layer**
		- Uses the keys established during handshake layer amongst other information to encrypt application-level packets.

### Handshake Layer Overview
- The TLS handshake is responsible for:
	- Establishing the cipher suite
	- Establishing the pre-master and  master secret
	- Permitting the resuming of sessions
	- Authenticating the identity of the server or client
	- Providing resistance against tampering and other attacks
![](Pasted%20image%2020260331202550.png)
#### Hellos
- The Client initiates by advertising its capabilities:
	- Supported TLS version
	- Random nonce(prevents replay)
	- Supported Cipher suites as a priority list(named combination of cryptographic algorithms, primitives, and parameters to be used for a TLS session)
		![](Pasted%20image%2020260331203721.png)
	- Extensions
	- Session ID - used for resuming previous sessions
- The server responds:
	- Selects a cipher suite from client's list it supports
	- Supplies it's own random nonce
	- Returns the session ID

#### Public Key Authentication
- The server sends its X.509 public key certificate containing:
	- The server's public key
	- Identity information e.g., domain name
	- Validity period
	- The CA's digital signature over the hash of all of this
- The client uses PKI to verify this certificate is genuine.
	 ![](Pasted%20image%2020260331204215.png)
	- Verify certificate by applying public key to signature, getting the hash back, and then computing hash over the contents and verifying there has been no modification. 
	- Walk up the 'Chain of trust' and verify the server's certificate using the public key of the issuer
	- Then take the certificate of the issuer, also supplied by the server, and verify that using the public key of the issuer of the issuer, who is typically a root CA which will have its public key in the local trust store.

#### ServerKeyExchange
- For ECDHE cipher suites (Elliptic Curve Diffie-Hellman Ephmeral), the server sends its Diffie-Hellman parameters:
	![](Pasted%20image%2020260331204727.png)
- The server signs these DH parameters with its private key and sends the signature and the DH parameters
- The client verifies using the public key which was verified in the prior step, which confirms the parameters came from the genuine server, which holds the private key.
	- This prevents a MITM attack where an attacker would intercept the DH parameters from the server and substitute their own deriving independent pre-master secrets for both the client and the server, giving them the ability to read and re-encrypt all traffic.
		![](Pasted%20image%2020260401012135.png)
	- This is because the attacker does not have the private key to sign the DH parameters they would be providing. 
- A fresh DH key pair is generated for each key-pair.

#### Certificate Request
- Optionally, the server can request a certificate from the client.
- This is mutual TLS, used in high security environments where both parties must authenticate.


#### ServerHelloDone
- Signals that there are no further messages to be sent from the server as part of the TLS handshake.


#### ClientKeyExchange
- The client sends its DH ephmeral public key, which under ECHDE is $A = a \cdot G$, aka $aG$ 
- Now both parties can compute the pre-master secret by applying $a$/$b$ to the ephemeral public key respectively.
- The pre-master secret goes through a Key Derivation Function to derive all session/symmetric keys from. 
- **Master secret = $= PRF(\text{pre-master secret, client random, server random})$
- **Symmetric key** = $=PRF(\text{Master Secret, further parameters})$
- Typically apply a hash function over the random values and the pre-master secret to obtain the master secret, and apply again over further parameters to obtain symmetric keys.
- The use of random numbers is to ensure session uniqueness; if the same pre-master secret was somehow negotiated in two different sessions (potentially by eavesdropping and starting own TLS handshake with same parameters) the resultant symmetric keys will be completely different.


#### Certificate & CertificateVerify
- If the server requested a client certificate, the client sends it here, along with a digital signature over all the handshake messages sent so far, proving it holds the private key corresponding to the public key in the sent signature.

#### ChangeCipherSpec
- A signal from each party that they are switching from the unencrypted handshake to the negotiated cipher suite. Everything after this point is encrypted with session keys, and sent AEAD-encrypted.

#### Finished
- Sent by both sides.
- Sends a $MAC(master\_secret, all\_handshake\_messages)$ over the entire plaintext handshake transcript from `Hello` onward.
- This finished message is sent encrypted by both sides, which verifies they both holds the same symmetric key too.
- Provides integrity; if the handshake or any of its contents were tampered with, the MAC will not match the locally computed MAC over the local transcript copy.
## PKI

### Why do we need PKI
- A server can send you one of two things
	1. A certificate - containing the server public key, identity info, a digital signature over the contents
	2. A digital signature only the server could have signed, verified using the prior obtained public key
- With no third party, there is a problem of circularity; we have no real reason to trust either as we use the certificate to verify the signature, but the certificate came from the server.
- MITM could intercept and substitute their own certificate and private key, and we would have no way to distinguish this from the real server.


### Digital Certificates
- Have a trusted third party to vouch for the binding between public key and an identity
- Takes the form of a third-party digital signature over the server's certificate (replacing its own signature)
- Certificates held in X.509 format defining the fields a certificate contains.

### Certificate Issuance
- A certificate comes into existence by a server generating a key-pair and creating a **Certificate Signing Request**
	![](Pasted%20image%2020260331212155.png)
	This CSR is signed by the server, proving they actually possess the corresponding private key. 
- A **Certificate Authority** uses the CSR to create and sign a certificate
	- First verifies the requester's identity
	- Then creates full certificate from CSR contents and computes a hash over all signature fields and signs the hash with the CA's private key (according to the `signature algorithm` field). 
	- Embeds the signature in the certificate and distributes it
		![](Pasted%20image%2020260328201229.png)

### Certificate Use
- The server presents the CA ordained certificate during the TLS handshake to verify the public key
- The client will receive the certificate and proceed to verify it, proving that the server's public key is valid. 
	- Can later prove server holds private key via signing of key exchange parameters.
1. Verify the certificate
	- Look up intermediate CA public key (usually also sent by server) 
	- Apply to signature in server certificate
	- Compute hash over the certificate fields
	- If they match, then continue up chain of trust
	- Look up root CA public key(local trust store)
	- Apply to signature in intermediate CA certificate
	- Compute hash over the certificate fields
	- If they match, then can be trusted
- The chain must end somewhere; the signature of a root CA simply comes from its own private key.
- Future signatures from the server can now be verified as we are sure we hold the public key.