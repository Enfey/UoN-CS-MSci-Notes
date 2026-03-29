## Signatures Overview
- A physical signature naturall serves as proof of authenticity, telling the recipient who sent the document. 
- Verification works by compring the signature against a known sample
- However not very robust - signatures can be forged
- This does not scale digitally, cannot efficiently verify millions of digital documents this way, motivating the need for a cryptographic alternative

## Electronic Signatures and Message Authentication
- The electronic equivalent works by computing a **signature function** over a message:  $$sig_k(x) = y$$
	- Where $x$ is the message, $k$ is a key, and $y$ is the signature.
- This computed binary signature is appended to the document
	![](Pasted%20image%2020260329181756.png)
- Verification asserts: $$(x, y) \rightarrow \mathrm{ver}_k(x, y) =
\begin{cases}
\text{True;} & y \text{ is valid} \\
\text{False;} & y \text{ is invalid}
\end{cases}$$
- Where the key $k$ is symmetric, the signature provides a few properties and namely does not provide another:
	- **Authenticity** - only someone with key $k$ could have produced the signature where $sig$ is an injection (one-to-one, map distinct messages onto unique codomain elements(but not all of them, so not bijection)) for key $k$ and message $x$ 
		![](Pasted%20image%2020260329182009.png)
	- **Integrity** - the signature is tied to the exact content of the message, so alteration to either the signature or the message contents will result in invalid $y$.
	- **Non-repudiation** - This is term meaning that there is an assurance that a party in a transaction cannot deny their authenticity or actions. Because $sig$ is predicated on a symmetric key $k$, either party could have generated the signature, with no way to attribute who sent it. Alice could deny sending a message by claiming Bob forged it, denying the action, meaning we do not have this property.


### Public Key Signatures
- Asymmetric cryptography grants us non-repudiation with digital signatures
- The scheme works as follows:
	- Alice has a private key $k_{prv}$ and a public key $k_{pub}$ 
	- Alice signs with her private key $s = sig_{k_{prv}}(x)$
	- Bob verifies with Alice's public key: $ver_{k_{pub}}(x, s) = true/false$ 
	![](Pasted%20image%2020260329183028.png)
#### RSA signatures
- RSA is the classic asymmetric key cryptographic algorithm to achieve cryptographic signing
- The setup is as follows:
	- Alice's private key: $k_{prvA} = (d)$
	- Alice's public key: $k_{pubA} = (n, e)$, sent to Bob
	- **Signing** $s = sig_{prvA}(x) \equiv x^d \ mod \ n$ 
	- Send $(x, s)$ to Bob for verification.
	- **Verification** $ver_{pubA}(x, s) = s^e \equiv \ mod \ n = x'$, check $x' = x$ 
	![](Pasted%20image%2020260329184415.png)
- The efficiency of signing $(x^d \ mod \ n)$ and verification $(s^e \ mod \ n)$ is governed by the use of the **square and multiply** algorithm, so efficiency depends on the size of the exponents.
- In practice, $e$ is kept small, commonly $65537 = 2^{16}+1$ which in binary is $10000000000000001_2$ which means that square and multiply requires very few multiplications
	- See more about the time complexity and choice of $e$ in [Lecture 10 - RSA](Lecture%2010%20-%20RSA.md)
	![](Pasted%20image%2020260329185415.png)
- This prioritises verification speed, which matters more because in practice, documents are often verified more frequently than they are signed.


## Signature Forgeries
- Note that we have used $x$ so far for the message, but $m$ is more common in the literature
- A **forgery** is producing a valid $(m, s)$ pair for a message $m$ that the legitimate signer actually signed
	- Note that simply replaying a previously observed $(m, s)$ pair doesn't count, since $m$ was genuinely signed by the author
- There are various severities of attack/forgery depending on the degree of control over the message $m$.

### Existential Forgery
- The attacker is able to create a valid message/signature pair $m$ with no constraints/ no ability to select $m's$ contents
- Given a public key $(n, e)$ an attacker:
	1. Selects random $s$ 
	2. Computes $m' = s^e \ mod \ n$ 
	3. Output $(m', s)$ as a forged pair.
- This naturally works because $s^e \ mod \ n$ is used to recover $m'$ definitionally.
	- Saying nothing about its validity, it is indeed a valid pair because we have no original message to check against as it has been completely synthesised.
- This is not very useful, but it is not completely harmless.
- $m'$ is essentially a random number with no semantic meaning, and the attacker has zero control over what $m'$ says.
- In poorly designed systems where messages aren't checked for structure of meaning, even a random valid pair could cause harm. 

### Selective Forgery
- Here the attacker fixes message $m$ first and attempts to find a valid signature $s$ to yield a valid message/signature pair where $m$ has been specifically chosen, and is usually meaningful. 
- It is a requirement that $m$ is fixed/chosen otherwise would just be an existential forgery.
- Computing a selective forgery (without a signing oracle) is equivalent to computing $s = m^d \ mod \ n$, which requires knowing the private key $d$. Without this key, the attacker would need to compute the factorisation of $n$ to compute $d$ under the modulo of $n's$ totient, which is computationally infeasible.
- Selective forgery is closely related to **chosen message attacks** where an attacker can query a signing oracle for signatures on chosen messages. 
	- The question becomes whether seeing valid signatures for arbitrary messages, helps forge a signature on the target $m$, revealing information about $d$. 
	- For RSA with strong key selection and padding, this is believed to be difficult.
### Universal Forgery
- The attacker is able to sign any message $m$ on demand and produce a corresponding signature $s$, yielding a valid message/signature pair under a private key $prvA$.
- For RSA, this almost certainly means they have recovered $d$; this is the only know way to produce valid signatures under a key for arbitrary messages. 
- This is equivalent to completely breaking RSA, and this level of forgery also implies/subsumes the other 2 levels.

### Malleability
> *A cryptosystem is deemed malleable if the attacker can modify the ciphertext or signature in a meaningful way and produce another valid ciphertext/signature predictably.*
- RSA signing is simply modular exponentiation
	- $s_1 = m^d_{1} \ mod \ n$ 
	- $s_2 = m^d_2 \ mod \ n$ 
- Exponentiation distributes over multiplication, meaning that:$$s_{1} \cdot s_{2} = m^d_{1} \cdot m^d_{2} \equiv (m_{1} \cdot m_{2})^d \ mod \ n = m^d_{3}$$
	- Multiplying the signatures is the same as the 3rd equation, this shows, meaning that you can produce a valid signature for the multiplication of those messages by just multiplying their signatures.
	- This yields the valid signature for $m_3$, whose contents can be acquired by performing just $m_1 \cdot m_2$.
- This means that you can transform a valid ciphertext/signature into another valid signature without knowing the key directly.
- We don't need to know exactly why this is the case, but this is more control for an attacker than we would like to have over a signature scheme.
- $m^3$ is random and will not yield a meaningful transaction, so this property is generally just another way to achieve existential forgery.
- This also affects RSA encryption.
## Padding
- The solution to the aforementioned weaknesses, particularly malleability and existential forgery, is to enforce strict formatting rules on the message $m$ prior to signing.
- If valid messages must follow a specific structure, then randomly generated forged messages from either malleability or the traditional method to achieve existential forgery are unlikely to conform to this structure
- We include a $y$ bit padding field for every message $m$
	![](Pasted%20image%2020260329194808.png)
- For a generated message $m' = s^e \ mod \ n$ it must contain all valid $y$ bits to be treated as a proper message.
- The probability that any $bit_y$ is a correct bit is $\frac{1}{2}$ which when multiplied $y$ times yields:
	![](Pasted%20image%2020260329195058.png)
	LIKELIHOOD OF SUCCESSFUL FORGERY IS THEREFORE $2^{-y}$ 
- Another way to prove this is that attacker's random $m'$ lands in a space of $2^n$ messages, of which only $2^{n-y}$ are valid messages because $y$ is fixed.
- Fraction that are therefore valid, as $n$ is irrelevant: $$\frac{2^{n-y}}{2^n} = 2^{-y}$$
## Hash-then-sign
- Rather than signing the message $m$ directly, the signer first computes a non-invertible $PRF$ over the message, called a **hash**:$$sig_{k_{prvA}}(m) \equiv H(m)^d \ mod \ n$$
- Verification recomputes the hash independently over $m$:$$ver_{k_{pubA}}(m) = s^e \ mod \ n \equiv H(m')$$ $$H(m) = H(m)'$$
- Checks that the hashes are equivalent, rather than the direct message contents.
### Benefits
- **Message size**
	- Raw RSA can only sign values smaller than $n$ (otherwise values would have multiple equivalents under a modulus and it would be impossible to decrypt them correctly).
	- Real messages are arbitrarily large.
	- Hashing compresses any message to a fixed size digest, ensuring that the message space is restricted to be $< n$ 
- **Existential forgeries become impossible**
	- Signatures are now computed over $H(m)$ rather than just $m$. 
	- With this, producing $m' = s^e \ mod \ n$ is no longer valid, as a valid signature now requires the message $m$, a hash over that message $H(m)$, and the signature over the hash of the message.
	- This means an attacker gets $s^e \ mod \ n = H(m')$ 
	- But they do not have the message $m$ that this hash belongs to.
	- Cryptographic hash functions are **preimage reistant**, meaning that given a hash, finding a message that produces it is infeasible.
	- Without a valid message $m$ when the receiver recomputes the hash, it will see they are not equivalent and discard the document.
- **Malleability**
	- The multiplicative property that:$$RSA(m_{1} \cdot m_{2}​)=RSA(m_{1}​)\cdot RSA(m_{2}​)$$ applied to hash messages would require:$$H(m_{3}​)=H(m_{1}​)⋅H(m_{2}​) \ mod \ n$$
	As RSA is no longer computed over the messages directly. Finding messages whose hash satisfies the above relationship and the implied relationship for solving the signature is infeasible for a good hash function.

The security of cryptographic signing is now essentially predicated not only on RSA, but arguments about hash functions. 

## PKCSv1.5
- PKCSv1.5 is a standard that defines a precise encoding for the padded message before signing. 
- 

## RSASSA-PSS
