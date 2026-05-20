## SSl/TLS
- Protocol that provides two fundamental security properties:
	- **Confidentiality**
	- **Authentication**
- Predecessor was SSL, had sec flaws
- Now standardised as TLS, anything older than 1.2/3 disabled on servers.

## TLS
- Arguably sits in layer 6 of the OSI model, the **presentation layer** which formats encrypts, compresses data from application layer
	- Sits among other protocols e.g., ASCII ensure data in usable, standardised format.
- TLS has two layers:
	1. **Record Layer**
		- Handles the encryption+integrity of data once session established
		- Takes application layer packets, encrypts them using symmetric keys agreed during handshake
	2. **Handshake layer**
		- Establish session keys + auth either party (server + optionally client)

### TLS handshake
#### Hello messages
- Client sends a `ClientHello` containing: TLS version it supports, a list of cipher suites supported, nonce
- Server responds with `ServerHello`, select cipher suite from client list, gen random number

#### Cipher Suite Negotiation
- A **Cipher Suite** is a named combination of cryptographic algorithms used for diff purposes e.g., TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256

#### Public key authentication
- Use of RSA for signing
	- RSA proof, commutativity of integer multiplication, does not matter, which order, keys are applied, need to keep secret $d$ safe
- Server sends an X.509 signature containing domain name, validity date, public key, crucially, digital signature from CA
	- CA takes cert contents, computes a hash, and signs with their private key
	- Client takes CA's public key and applies it to their signature recovering the hash, then recompute same hash to verify, ensuring contents (including server public key) are legit. 

#### Key Exchange
- Elliptic Curve Diffie-Hellman Ephmeral
- Do freshly for every handshake, generate temp pre-master secret.
- Server signs DH parameters now that public key trusted to avoid MITM, and sends the signatures, and the DH parameters.
	- Apply public key, check params match


#### Key Derivation
- Once both side have premaster secret, need to derive symmetric keys from pre-master secret
- Hash function, takes pre-master secret, the 2 random hello values, computes master secret and symmetric key. 
	- Master secret = PRF(pre-master secret, client random, server random)
	- Symmetric key = PRF(master secret, some more params)
- Random numbers ensure session uniqueness; on off chance that same pre-master secret computed via ECDHE, the random values would ensure be different.
- Often derive multiple keys from master secret. 

#### Symmetric encryption
- Use AES in GCM with 128 bit key for data
- Use GCM for auth/integrity component that its tag provides which hardens against ciphertext alteration/malleability
- Use of AES due to speed; 128 bit 40% faster than 256 bit.


#### Finished messages
- After all key exchange and authentication, both sides send a `Finished` message
- This is a hash of the entire handshake transcript, ensures both sides seen same version of events, provides a **further integrity** check. 
- Record layer takes over now and symmetric encryption is used for application data.

## TLS 1.3
- Significant changes to protocol:
	- **Handshake improved**
		- Much faster, and embeds key exchange into hello messages
		- Client picks most likely KE algo, includes public value immediately, and if server supports, responds immediately with EVERYTHING in the second message, means that encryption can begin after second message in exchange.
		- If the guess of KE algo incorrect, server sends retry, specifiying which algo to use.
	- **Cipher removal**
		- 3DES, RSA key transport, static DH, all removed, eliminating potential for TLS 1.3 to be negotiated/downgraded
	- **Key exchange and authentication separated from ciphers**
		- Cipher only specifies encryption now, easier negotiation + keep track of
	- **Wider use of extensions**
		- 0-RTT resumption:
			- Allows client connecting to known server to send application data in first message using pre-derived key from prior session
			- Weaker security means vulnerable to replay attacks
				- No new random computed to define a new session, so all prior packets are valid and can be resent
				- If SK compromised, can decrypt prior data; more impactful, as will likely have been using that key consistently since session initiated.


### TLS vulnerabilities
- **Majority are implementation based** e.g., buffer overflow
- **Protocol downgrade**
	- Interfere with cipher suite negotiation in ClientHello, forcing both to agree on weak cryptography that can be broken
	- TLS 1.3 circumvents
	- TLS 1.2 that advertise weak suites still vulnerable.
- **MITM**
	- Countered by public key authentication as attacker does not have private key needed to sign DH parameters. 


# Public Key Infrastructure
## Why we need
- Servers can send two things in TLS:
	1. **Certificate**
		Contains public key, domain name, digital signature over the contents
	2. **Digital signature**
		Digital signature that only the server could have signed, verified using obtained public key
- Circular issue we use the certificate to verify the signature, but the certificate came from server
	- MITM could intercept, supply own certificate and public key, would have no way to distinguish
- Need trusted third party.

## Digital Certificates and X.509
- Solution = trusted third party to vouch for binding between **identity** and **public key**
- Takes form of third-party digital signature over the server certificate's contents
- Certificate's held in X.509 format-defines structure and fields certificate must contain

### Certificate Issuance
- Certificate comes into existence by generating a key pair and creating a **Certificate Signing Request**
	![](Pasted%20image%2020260328201223.png)
	CSR signed by server's own private key. This is to prove to the CA that the requester actually posses the private key corresponding to the public key in the CSR otherwise anybody could just submit a CSR impersonating the server.
- A **Certificate Authority** verifies the requester's identity, verifies public key, verifies the server controls the domain being certified.
- It then creates the full certificate from CSR contents, computes a hash over the certificate fields, and signs the hash with its private key to produce the signature, then embedded in the certificate. 

## Certificate use
- Server presents CA signed cert during tTLS
- Client receives (and if TLS 1.3, signed DH params too)
- Verifies it with CAs public key, shows that server's public key is owned by it, and its private key is valid regarding the DH signature.
	1. **Verify signature**
		- Look up CA public key in local trust store
		- Apply public key to signature, yield the hash
		- Recompute hash and check match
	2. **Verify DH signature**
		- Apply public key to signed DH params
		- Compute hash over them too
		- If match, server holds corresponding private key. 
- Could not alter reliably - would have to alter the hash and cert contents in predictable manner

## Chains of trust
- Machine cannot store cert for every server, there are millions of them
- Maintain small number of **root certificates** belonging to highly trusted CAs
- Root CA is top-level entity in PKI hierarchy; issues certs to intermediate CAs, who then issue to leaf CAs
- When leaf CA presented, find issuer, is intermidiate, find issuer, get public keys along way, keep going until find root cert that matches one in trust store, then walk back along and verify. 

### Who manages root certs
- OS - Apple, wWindows
- Each one runs a **root certificate program** - set of requirements CA must meet to be eligible to be a root CA
- Security of TLS rests on trusting these vendors to audit CAs properly, and trusting the CAs to behave properly and securely

### Limitations of PKI
- **TLS inspection possible**
	- When root cert added to trust store, blanket trust possible
	- If a trust store contains compromised root cert, and someone can run a proxy between your machine and internet, have problem
	- Proxy completes normal TLS handshake with the internet and with you, MITM, and decrypts its traffic and forwards it to you.
		- It can do this because it presents a false certificate for the internet serer signed by the compromised root CA private key, which is then verified using the public key you have, so it looks legit, and proxy can read and decrypt on both sides. 
- **Private key compromised**
	- If priv key stolen, must be revoked, clients need to be told to stop trusting associated cert
	- 2 ways: 
		1. **Certificate Revocation lists**
			- CA periodically, publish signed list of revoked certificate serials
			- Client download, verify, can be large, and can be window after revocation where cert appears valid
			- DoS attack on URL, clients skip check entirely
		2. **Open certificate status protocols**
			- Client sends cert serial num to CA's OSCP
			- CA responds with good/unknown/revoked
			- Response is signed and cached
			- Privacy: CA learns which certs are being checked
			- OSCP server must be available 