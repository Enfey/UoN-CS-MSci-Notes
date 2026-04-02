## RSA
- Asymmetric encryption algorithm introduced in 1997, and is the most popular public key algorithm in the world.
- RSA uses two mathematically linked keys - a public and a private key. What the public key encrypts, only the private key can decrypt. 
- This solves a fundamental problem with symmetric cryptography; we no longer need to share a secret key in advance. 
- Encryption performed by the public key can only be reversed using the private key:
	![](Pasted%20image%2020260309173007.png)
	- Server maintains a key pair, and shares public key with anyone who wants to communicate. Clients encrypt messages via the server's public key, but only the server's private key can decrypt it. 
	- Historically, was used to encrypt symmetric session keys to send to the server e.g., an AES key. The server decrypts with its private keys, and now both sides share a secret symmetric key. 
	- The server can also use its private key to sign a document, and send both the document and the signature to the client, the client uses the server's public key to verify the signature has not been forged (confirms hold private key, and message hasn't been tampered).

## Euler Totient Function
- Integers $a$ and $m$ are coprime if they do not share a divisor (except 1):$$gcd(a, m) = 1$$
- The **Euler Totient $\Phi$** is the number of integers in $Z_{m} = \{1, \dots, m-1\}$ for which $gcd(a, m) = 1$ 
	- Counts how many integers from $1$ to $m-1$ are coprime with $m$.


### Integer Factorisation
- Any integer can be expressed as the multiplication of a list of prime numbers
- For example:
	$103284720$ = $2 \times 2 \times2\times2\times3\times3\times5\times7\times9\times11\times23 = 2^4 \times 3^2 \times 5 \times 7 \times 11 \times 23$
- The greater the value, the more computationally expensive this gets. 
### Calculating $\Phi(n)$ 
- The totient is much easier to calculate given the prime factorisation of $n$
	- $m = p_1^{e_1} \cdot p_2^{e_2} \cdot \dots \cdot p_n^{e_n}$ 
	- $\Phi(n) = \prod^{n}_{i=1}(p^{e_{i}}_{i} - p^{e_{i-1}}_{i})$ 
- For example, take $m = 240$ 
	- $240 = 24 \times 10$ 
	- $24 \times 10 = 12 \times 2 \times 2 \times 5$
	- $= 3 \times 2 \times 2 \times 2 \times 2 \times 5$ 
	- $= 2^4 \times 3^1 \times 5^1$ 
	- $\Phi(240) = (2^4 - 2^3) \cdot (3^1 - 1) \cdot (5^1 - 1)$ 
		- $1$ because $x^0 = 1$ obviously
	- $=(16 - 8) \cdot 2 \cdot 4 = 64$ 
	- Thus, there are $64$ numbers between $1$ and $239$ that share no common factors with $240$. 

### Calculating $\Phi(p)$ for primes
- For a prime $p$, it only has one prime factor - itself, with exponent $1$.
- Therefore $\Phi(n) = (p^1 - p^0) = (p-1)$ 
- This is similar for semiprimes, that is, numbers formed by multiplying two primes, $n = p \cdot q$ 
	- $\Phi(n) = (p^1  - p^0) \cdot (q^1 - q^0) = (p-1)(q-1)$ 
	- This is very easy to calculate if you know $p$ and $q$, but very hard if you only know $n$.


### Fermat's Little Theorem
- Fermat's little theorem states that for some prime $p$, and any integer $a$: $$a^{p-1} \equiv 1 \ (mod \ p)$$
- Raising any integer and performing modulo results in $r =1$.

### Euler's Theorem
- Euler generalised Fermat's result to work for any modulus, not just primes: $$a^{\Phi(m)} \equiv 1 \ (mod \ m)$$
- Where the only condition needed is $gcd(a, m) = 1$ 
- This works for any integer ring $Z_m$ 
	- A ring extends a group by adding a second operation, the ring with $+$ forms an abelian group, and forms a semi-group under multiplication
		- Multiplication is associative, but it may not be commutative, and multiplicative inverses may not exist (if both were true, this would constitute a **Field**).
	- Multiplication distributes over addition, but addition does not distribute over multiplication, though this is just generally the case anyway. 
		![](Pasted%20image%2020260309182109.png)
		![](Pasted%20image%2020260309182115.png)


## RSA - Key Generation
1. Choose two large primes, $p$ and $q$ 
2. Calculate the modulus $n = p \cdot q$ 
3. Compute the totient $\Phi(n) = (p - 1)\cdot(q - 1)$ 
4. Choose a value $e \in \{2, \dots, \Phi(n)-1\}$ where $gcd(\Phi(n), e) = 1$ 
5. Compute $d$ where $d \cdot e \equiv 1 \ (mod \ \Phi(n))$ 

### Example
- $p = 17, q = 11$ 
- $n = 187$
- $\Phi(n) = 160$, 160 numbers between $1$ and $186$ that are coprime with $187$ 
- $e = 3$, $gcd(160, 3) = 1$,
	See previous lecture for EA.
- $d = 107$, $107 \cdot 3 = 321$, $321 (mod \ 160) = 1$ 

The public key is $n$ and $e$, and the private key is $p, q, (\Phi(n)), d$, but we just take $d$.
	NOTE: to break RSA, must compute totient, and need p and q for this, given only $n$, computing the totient is equivalent to factoring $n$ for all primes. So breaking RSA amounts to factoring $n$, which for large primes is impossible. After finding $p$ and $q$, then $d$ is computed via EEA.

## RSA - Encryption
- Now that we have a a public key $(3, 187)$ and a private key $107$ 
- Encryption and decryption is performed by:
	- $x^e \equiv y \ (mod \ n)$ 
	- $y^d \equiv x (mod \ n)$ 
- For example:
	- $x = 74$
	- $x^3 = 182 (mod \ 187) = y$
	- $y^{107}= 182^{107} (mod \ 187) = 74$
- The proof is by magic!:
	- $d \cdot e \equiv 1 (mod \ \Phi(n))$ 
	- $e \cdot d = 1 + k \cdot \Phi(n)$
	- $x^{e \cdot d} = x^{1 + k \cdot \Phi(n)}$ 
		$= x \cdot x^{k \cdot \Phi(n)}$ 
		Rest of proof, understand, but don't really need to know:
	 ![](Pasted%20image%2020260309185703.png)
	There is of course the assumption that $gcd(x, n)$, that is the plaintext and the semiprime modulus are coprime. In practice this is never usually a problem, because $p$ and $q$ are enormous primes. The probability that a random message happens to be a multiple of either is astronomically small. 


### Why RSA is Secure
- The attacker has the public key $(n, e)$ and the ciphertext $y$.
- To recover $x$ they need to solve: $$y^? \equiv 1 (mod \ n)$$
- They need to find $d$. Computing $d$ requires solving $e \cdot d \equiv 1 (mod \ \Phi(n))$, which requires knowing the other unknown, $\Phi(n)$, $\Phi(n) = (p-1)(q-1)$, we need to know $p$ and $q$ to determine the totient for $n$, as these are the factors of $n$.
- For a 2048 bit $n$, computing $p$ and $q$ to factor $n$ is computationally infeasible, which means one cannot recover $d$ and thus the plaintext. 

## Exponentiation
- RSA requires computing things like $x^e$
- The naive approach would be to multiply $x$ by itself $e$ times. This works where $\Phi(n)$ is small, and the integer set we choose $e$ from is therefore also small.
- This is impractical where exponents can be thousands of bits long, for thousands of bits long $n$.
- For example:
	- $x^8$ requires $7$ multiplications
		![](Pasted%20image%2020260309191911.png)
	- But it can be shortened by repeatedly squaring:
		- $x \cdot x = x^2$ 
		- $x^2 \cdot x^2 = x^4$ 
		- $x^4 \cdot x^4 = x^8$ 
		Only 3 operations!.
### Square and Multiply
- Introduce multiplications, rather than just using squaring too. 
- The target exponent is represented in binary
	![](Pasted%20image%2020260309193551.png)
- And we carry out square and multiply operations such that it matches the exponent. 
	![](Pasted%20image%2020260309193703.png)

### Computational Complexity of Large Bit Exponentiation
- For a 2048 bit RSA key, the exponent is on the order of $2^{2048}$, that is $x^{2^{2048}}$ 
	- For every 0, have to do a square, for every 1 have to do a square then a multiply.
	- So for $T$ bits, roughly $1.5T$ operations in theory, which is much more feasible via use of square and multiply. 


### 65537
- It is the largest known number of the form $2^{2^{n}}+1$ 
- Primes of this form are known as Fermat Primes.
- Specifically takes the representation $2^{2^{4}} = 2^{16} +1$
- This makes its binary representation $10000000000000001$, which means that there are 16 squarings, and only 2 multiplications, making it just 18 operations to encrypt.
- Standard RSA public exponent $e$ for two reasons:

## Signing
![](Pasted%20image%2020260309200030.png)
