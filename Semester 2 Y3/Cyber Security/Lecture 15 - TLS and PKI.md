## SSL/TLS history and overview
- Transport Layer Security (TLS) is a protocol that provides two fundamental security properties for network sessions:
	- **Confidentiality**
	- **Authentication**
- Its predecessor was Secure Socket Layer (SSL) but had serious security flaws. 
- Had numerous iterations and was eventually standardised as TLS, with TLS 1.2 and 1.3 being the acceptable versions, with anything older being disabled on servers. 

## TLS
- Arguably sits in layer 6 of the OSI model, the **presentation layer** which formats, encrypts, and compresses data coming from the application layer 
	- Sits amongst other protocols/formats like MIDI, ASCII, which are responsible for ensuring data is in a useable, standardised format.
- TLS itself has two internal sub-layers:
	- **Record Layer**
		- Handles the encryption and integrity of actual data once a session is established
		- Takes application-layer packets and encrypts them using symmetric keys agreed during the handshake, much like IPSEC.
	- **Handshake Layer**
		- Used to establish session keys, as well as authenticate either party (verify the server, and optionally the client).

### TLS Handshake
1. **Client and server say Hello**
2. **Pair agree on cipher suite**
3. **Public key verification**
4. **Key exchange**
5. **Final checks**
6. **Send application data**
![](Pasted%20image%2020260328171516.png)

#### Hello messages
- The client sends a `ClientHello` containing: the TLS version it supports, a list of cipher suites it can use, and a randomly generated number 
- The server responds with a `ServerHello`, selecting a cipher suite from the client's list , and supplies its own random number
- These random numbers are to be used later in key derivation.

#### Cipher Suite negotiation
- A **Cipher Suite** is a named combination of cryptographic algorithms, primitives, and parameters to be used for the whole session:
![](Pasted%20image%2020260328172004.png)
- **ECDHE**
- **RSA**
- **AES_128_GCM**
- **SHA256**
#### Public key authentication
- Use of RSA for cryptographic signing. 
- Note that, given the proof of RSA, and the commutativity of integer multiplication (as they appear as powers of the plaintext), that the order in which the public and private key are applied do not matter, either will recover the original, but it is the secrecy of $d$ that makes RSA secure.
- The server sends an X.509 signature containing its public key, validity dates, domain name, and crucially, a digitial signature from a CA.
- The CA took all certificate contents e.g., domain name, public key, etc, and computed a hash over it, and then signed it using the CA's own private key $d$ 
- The client takes the CA's PUBLIC key (already stored locally) and applies it to the signature recovering the hash, then computes hash over the plaintext certificate contents sent by the server and verifies they match ensuring the contents (including the server public key) are legitimate.

#### Key Exchange
- Elliptic Curve Diffie-Hellman Ephemeral
- Ephemeral means we do this fresh for every handshake - generate temporary *pre-master secret* for every handshake
- The server actually signs DH parameters and sends the signature, and the DH parameters
- The client can verify using the public key, and check the signature and the DH parameters match, as the public key was verified in the previous step.
- This prevents MITM attack where an attacker would intercept the DH parameters from the server and substitute their own deriving independent pre-master secrets from both the client and server, giving them the ability to read and re-encrypt all traffic as if nothing had changed.
	![](Pasted%20image%2020260328175921.png)
	- Cannot simply do this, as don't have the private key $d$ to be able to sign the DH parameters.

#### Deriving the Symmetric keys
- Once the DH exchange is complete and both sides have the pre-master secret, they need to derive actual symmetric keys from the pre-master secret.
- A hash function (namely SHA-256 as named in the cipher suite) takes the pre-master secret, and the `CLIENT_RANDOM` and `SERVER_RANDOM` values from `CLIENT_HELLO` and `SERVER HELLO` to compute the symmetric session key
	1. **Master Secret** $= PRF(\text{pre-master secret, client random, server random})$
	2. **Symmetric key** $=PRF(\text{Master Secret, further parameters})$
- The use of the random numbers is to ensure session uniqueness; on the off-chance that the same pre-master secret was somehow negotiated in two different sessions, the random numbers mean the derived symmetric keys will be completely different. 
- Compute a master secret first, then a symmetric key as in practice we often need to derived multiple keys from the same handshake. 

#### Symmetric Encryption
- Use AES in GCM with 128 bit key for the actual data.
- Use GCM for the authentication/integrity component that its tag provides which hardens against ciphertext alteration/high malleability primitives.
- Use of AES due to speed, 128 bit 40% faster than 256 bit.
#### Finished messages and Application data
- After all key exchange and authentication, both sides send a `Finished` message
- This is a hash (digest) of the entire handshake transcript
	- That is, the hash computed over every message exchanged from `Hello` onward
- Provides an:
	1. **Integrity Check**
		- If the handshake, or any of its contents were tampered with in transit (and this wasn't bidirectional, as it usually is impossible to do so) then the hash will not match the locally computed hash over the transcript when decrypting
- After the Finished messages, the handshake layer steps aside, and the record layer takes over entirely. 
- All subsequent communication is:
	- Plaintext -> AES-128-GCM encryption -> ciphertext on wire and vice versa for decryption, over application data.


### TLS 1.3
- Makes significant changes to the protocol:
	- **Major Handshake Improvements**
		- Faster handshake
		- Embeds much of the key exchange into the hello messages
		- The client picks the most likely key exchange algorithm and includes its DH public value immediately.
		- If the server supports it, it can respond with everything needed in one message, with encryption beginning after the second message in the exchange.
		- If the guess of key exchange algorithm is incorrect, the server sends a HelloRetryRequest specifying which algorithm to use and the client then retries.
	- **All ciphers removed except for modern AEAD**
		- 3DES, CBC mode, RSA key transport, static DH etc all removed, eliminating the potential for a TLS 1.3 connection to be negotiated as a weak cipher suite regardless of server-side misconfiguration, eliminating entire categories of attack.
	- **Key exchange and authentication separated from ciphers**
		- Cipher suite only specifies encryption in TLS 1.3:
			- `TLS_AES_128_GCM_SHA256`
		- Simpler negotiation and easier to keep track of.
	- **Wider use of extensions**
		- 0-RTT below is implemented via extensions, included in `ClientHello`, same with `key_share` which tells the server to send the DH public key immediately.
	- **0-RTT-Resumption**
		- Allows client reconnecting to known server to send application data in very first message, using pre-derived key from the previous session, giving zero overhead for reconnections
		- Weaker security guarantees and is potentially vulnerable to replay attacks.
			- May process multiple requests e.g., transfer 100 under HTTPS as there is no new random that has been computed to define a new session, so all prior packets are valid and can be resent.
			- If the PSK is compromised, all recorded early data dating back to when the session was initiated can be decrypted.

### TLS Vulnerabilities
- Majority of TLS problems are not theoretical, but implementation based
	- See heartbleed from prior lecture
- Protocol downgrade attacks are possible by interfering with the Cipher suite negotiation in the ClientHello message, forcing both parties to agree on deliberately weak cryptography e.g., 512-bit RSA keys and 512-bit DH parameters are both breakable.
	- TLS 1.3 eliminates this by removing all weak options, but TLS 1.2 servers that still advertise these suites remain vulnerable
- MITM is countered by the public key authentication mechanism - the CA certificate chain combined with signed DA parameters. Authentication is not optional in TLS.

## Public Key Infrastructure
### Why do we need PKI
- A server can send you one of two things (TLS):
	1. The certificate - contains the public key, the identity info, and a digital signature over the contents
	2. A digital signature that only the server could have signed, verified using the prior obtained and verified public key.
- The problem is circular in that we are using the certificate to verify the signature, but the certificate itself came from the server. 
- MITM could intercept the connection, substitute their own certificate and private key, and would have no way to distinguish this from the real server. 
- PKI breaks this circularity by introducing a trust third party whose trustworthiness comes from outside the connection itself. 

### Digital Certificates and X.509
- The solution is to have a trusted third party to vouch for the binding between a **public key** and an **identity**
- This takes the form of third-party digital signature over the certificate contents - this replaces the server's own signature over the certificate contents.
- Certificates are held in X.509 format, which defines the structure of fields the certificate must contain:
	![](Pasted%20image%2020260328201015.png)

### Certificate Issuance
- A certificate comes into existence by a server generating a key pair and creating a **Certificate Signing Request (CSR)**:
	![](Pasted%20image%2020260328201223.png)
	This CSR is signed by the server's own private key. This is to prove to the CA that the requester actually posses the private key corresponding to the public key in the request. Without this, anyone could submit a public key and get a certificate for it.
- A **Certificate Authority** uses the CSR to create and sign a certificate
- The CA first verifies the requester's identity, verifies the server controls the domain/organisation being certified.
- It then creates the full certificate from the CSR contents, computes a hash over all certificate fields according to `signature algorithm`, signs the hash with the CA's private key to produce the signature, and embeds the signature in the certificate and distributes the certificate.
	![](Pasted%20image%2020260328201229.png)

### Certificate Use 
- The server presents the CA ordained certificate during the TLS handshake to verify the public key. 
- The Client will receive the certificate (and if using TLS 1.3, the DH signature in the same step) and proceed to verify it, proving that the server's public key is valid (and its private key is valid regarding the DH signature):
	1. Verify the certificate
		- Look up CA public key from local trust store
		- Apply to the signature in the certificate
		- Compute hash over the certificate fields
		- If they match, then the certificate is genuine and unaltered.
	2. Verify the servers DH signature:
		- Know the public key is valid, so apply it to signed DH parameter hash
		- Compute hash over DH parameters
		- If they match, then the server holds the corresponding private key. 
- Could not alter reliably - would have to alter the hash and the certificate contents in a predictable manner which is essentially impossible.

### Chains of Trust
- Our machine cannot store a certificate for every server, there are millions.
- We instead store a small number of **root certificates** belonging to highly trusted CA's.
- A root CA is a trusted, top-level entity in the PKI hierarchy, responsible for issuing self-signed root certificates that act as the ultimate trust anchor and end of the chain of trust in PKI infrastructure.
	- Root CA private keys are extraordinarily sensitive, cannot sign everything with it, if it were ever compromised, every certificate signed by it would be untrustworthy
![](Pasted%20image%2020260328202930.png)
1. Look at server.com certificate issuer field
	- Need to verify this certificate
2. Take GeoTrust SSL CA - G3 public key
	- Verify its signature on the Server.com certificate to conform this intermediate CA signed server.com's certificate with its private key.
3. Look at GeoTrust SSL CA Issuer field
	- Need to verify this certificate
4. Take GeoTrust Global CA's public key from the local trust store
	- Verify its signature on the intermediate signature to confirm the root CA signed the intermediate CA certificate with its private key.
5. Server.com is trusted
- The trust in root certificates comes entirely from the OS vendor's decision to include them
	- The chain must end somewhere, and cannot continue forever
	- The signature of a root CA simply comes from its own private key.
- The server provides both the 'end-entity' and intermediate certificates during the TLS handshake, but the root certificate lives on your machine. 
- USE THE PARENTS PUBLIC KEY TO VERIFY THE CHILDS CERTIFICATE :D

### Who manages Root Certificates?
- Major OS vendors:
	- Apple for iOS and OS X
	- Microsoft for windows
	- Mozilla for Firefox and Linux
- Each of these vendors runs a **root certificate program** - a set of requirements a CA must meet to be included, which includes auditing requirements, security practices, and policies about what kinds of certificates the CA is permitted to issue. 
- The security of TLS therefore rests on trusting these vendors to run their programs well, and trusting the included CAs to behave honestly.


### Limitations of PKI
- **TLS inspection is possible**
	- When a root certificate is added to trust store, there is blanket trust for all of these CAs
	- If a trust store contains a malicious/compromised root certificate, and someone can run a proxy between your machine and the internet, then we have a problem!
	- The proxy completes a normal TLS handshake with the internet server, and can now decrypt its traffic and forward it to you.
	- The proxy also completes a TLS handshake with you, but presents a false certificate for the internet server signed by the compromised root CA private key, which is then verified using the public key of the root CA.
	- The proxy can now decrypt and encrypt on both sides and forward the traffic; seeming like nothing is wrong, but is able to read all traffic.
		![](Pasted%20image%2020260328205152.png)
- **Private key compromise**
	- If a private key is stolen or a certificate is misissued, it must be revoked - clients need to be told to stop trusting it before it expires naturally
	- 2 ways of doing this:
		1. **Certificate Revocation Lists**
			- CA periodically publishes a signed list of revoked certificate serials
			- Clients must download and check during verification of signatures.
			- Can be large, and can be window after revocation where the certificate still appears valid as it is updated only periodically
			- DoS attack on this URL, clients will skip check entirely.
		2. **Open Certificate Status Protocol**
			- Client sends certificate serial number to CA's OSCP responder
			- CA responds: good/revoked/unknown
			- Response is signed and can be cached for a period
			- Privacy issue: CA learns which certificates are being checked, effectively tracking browsing
			- OSCP server must be available.
	- Many clients do not properly check either.
