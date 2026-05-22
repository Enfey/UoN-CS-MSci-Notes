## Signatures and Message auth
- Compute **signature** over a message to give some authenticity to it and its contents, usually that it has not been tampered with. 
- $sig_k(x) = y$ 
- The computed binary signature is appended to the respective document
-  Verification asserts: $$(x, y) \rightarrow \mathrm{ver}_k(x, y) =
\begin{cases}
\text{True;} & y \text{ is valid} \\
\text{False;} & y \text{ is invalid}
\end{cases}$$
- The signature provides a few properties and namely does not provide another:
	- **Authenticity**
		- Only someone with the key $k$ could have produced the signature, where $sig$ is an injection(one-to-one, map distinct messages onto unique co-domain elements, but not all of them, so not bijection) for a key $k$ and message $x$ 
		![](Pasted%20image%2020260329182009.png)
	- **Integrity**
		- The signature is tied to the exact content of the message; if the message is altered, the signature will be invalid as reverifying the message will result in invalid $y$ 
	- **Non-repudiation**
		- Assurance that a party in a transaction cannot deny their authenticity or actions
		- Sig predicated on key $k$, (where here, we pretend the key is symmetric)
		- There would be no non-repudiation because either party could have generated the signature, with no way to attribute who sent it. Alice could deny sending a message by claiming Bob forged it. 
		- We do not have this properrty without the use of public keys



### Public Key Signatures
- Asymmetric cryptography grants us non-repudiation with digital signatures
- Alice has private key and public key
- Alice signs with her private key, and bob verifies with her public key, via the verification algorithm

#### RSA signaturs
- Setup is as follows:
	- Alice's private key: k_prv(d)
	- Alice's public key: k_pub(n, e)
	- Signing: sig_{prvA}(x) = x^d mod n
	- Send (x, s) to bob for verification, along with public key
	- check x' = x
- The efficiency of signing and verification is governed by the use of the square and multiply algorithm, in practice e is kept small, with a good binary representation to minimise either of the operations whilst allowing large enough primes to be chosen
- This prioritises verification speed over signing however, but that is fine because we are usually verifying, not signing. 


## Signature forgeries
- Used x so far, now use m
- A forgery is a valid $(x, m)$ pair for a message $m$ that the legitimate signer did not sign
	- Replkay doesn't count as it was actually signed
- There are various severities of attack classes depending on the degree of control over the message $m$


### Existential Forgeries
- The attacker can create a valid m, s with no ability to select m's contents
- Given a public key, the attacker
	1. Selects random $s$ 
	2. Computes $m' = s^e \ mod \ n$ 
	3. . Output $(m', s)$ as a forged pair.
- $s^e \ mod \ n$ is used to recover $m'$ definitionally.
- Says nothing about validity; it is indeed a valid pair because we have no original message to check against, completely synthetic. 
- Not very useful, but not completely harmless
- m' is essentially a random number with no semantic meaning, and the attacker has zero control over what m' says. 
### Selective Forgeries
- Here the attacker fixes $m$ first and attempts to find a valid signature $s$ that corresponds to $m$
- It is a requirement that $m$ is fixed/**chosen**, otherwise would just be existential forgery
- Computing a selective forgery is equivalent to computing the actual signature under the private key, whhich without, would need to factor n to get the totient, computationally infeasible.
- Selective forgery is closely related to **chosen message attacks** where an attacker can query a signing oracle for signatures on chosen messages. 
	- The question then becomes whether seeing a valid signature for arbitrary messages helps forge a signature on the target $m$; do they reveal anything about $d$?
	- Believed to be difficult.


### Universal forgery
- The attacker is able to sign any message $m$ on demand and compute a corresponding signature $s$ yielding a valid m/s pair.
- For RSA almost always means they have recovered $d$; only way to produce valid signatures under a key for arbitrary messages. 
- Equivalent to completely breaking RSA and subsumes the other 2 levels. 

## Malleability
- A cryptosystem is deemed **malleable** if the attacker can modify the ciphertext or signature in a meaningful way and produce another valid ciphertext/signature predictably. 
- RSA signing is modular exponentiation
- Exponentiation distributes over multiplication.
	- This means that one can produce a valid signature for the multiplication of messags by simply multiplying their signatures.
	- This means you can transform a ciphertext/signature under RSA into another valid signature without knowing the key directly.
	- This is more control than we would like an attacker to have over a signature scheme
	- m3 is random and will not yield a meaningful transaction, this is just another way to achieve existential forgery.

### Padding
- The solution to the aforementioned weaknesses, particularly malleability and existential forgeryo9   