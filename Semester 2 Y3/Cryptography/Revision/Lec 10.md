### RSA
- **Asymmetric** encryption algorithm, uses 2 mathematically linked keys, public and private key. 
- What the public key encrypts, only the private key can decrypt
- This solves fundamental problem with symmetric cryptography, key management becomes much easier
	- No longer need to share secret key in advance
- Encryption performed by the public key can only be reversed by the private key and vice-versa
	![](Pasted%20image%2020260309173007.png)
	- Server maintains key pair, shares public key, client encrypts with server public key, only server private key can decrypt
	- Historically used for key transport e.g., to send AES key to server
	- Server also use private key to sign certificates and send signature and X.507 signature to client, when pub key verifies (via chains of trust) confirms that their public key and private key is valid

### Euler Totient function
 - Integers $a$ and $m$ are coprime if they do not share a divisor such that $gcd(a, m) = 1$ 
 - The **Euler Totient** $\Phi$  is the number of integers in $\mathbb{Z}_m = \{1, ..., m-1\}$ for which $gcd(a, m) = 1$ 
	 - Counts how many integers from $a$ to $m-1$ are coprime to $m$ 
		 - Note that here the integers form a ring; we assume that the multiplicative group is incomplete as not all elements have a multiplicative inverse.

### Integer Factorisation
- Any integer can be expressed as the multiplication of a list of prime numbers
- $103284720$ = $2x2x22x3x2....$ 
- The greater the value, the more computationally expensive this gets


### Calculating $\Phi(n)$ 
- The totient is much easier to calculate given the prime factorisation of $n$
	- $m = p_1^{e_{1}} \cdot p_{2}^{e_{2}} \cdot \dots p_{n}^{e_{n}}$ 
	- $\Phi(m) = \prod^n_{i=1}(p_i^{e_i} - p_{i}^{e_{i-1}})$ 
- For example take $m = 240$ 
	$240 = 10 \times 24$ 
	5 x 2 x 12 x 2
	5 x 2 x 3 x 2 x 2x2
	$5 \times 3 \times 2^4$ 
	$\Phi(M) = (5 - 1) \cdot (3 - 1) \cdot (2^4 - 2^3)$ 
	$=(16-8) \cdot n \cdot 4 = 64$ 
64 numbers between 1 and 239 that share no common factors with 240. 


DO THE PRIME FACTORS, THEN JUST MULTIPLY THEM TOGETHER BUT BRACKET THEM AND MINUS ITSELF AND ONE OF ITS POWERS.



### Calculating $\Phi$ for primes
- $\Phi(p)$ much easier, there is only one prime factor, itself and 1
- Therefore $\Phi(p) = (p - 1) \cdot (1-1) = (p-1)$ 
- This is similar for semiprimes, that is, numbers formed by multiplying two primes $n = p \cdot q$ 
	- $\Phi(n) = (p^1 - p^0) \cdot (q^1 - q^0) = $(p-1)(q-1)$ 
	- This is very easy to calculate if you know $p$ and $q$, but **very difficult** if you only know $n$ as you do not know the prime factors to be able to construct this. 

### Fermat's little theorem
- States that for some prime $p$ and any integer $a$:
	$a^{p-1} \equiv 1 \ (mod \ p)$ 
- Raising any integer to $p^{-1}$ and then performing modulo under that same integer yields the neutral element.

### Euler's theorem
- Generalised fermat's result, work for any modulus, not just primes:
	$a^{\Phi(m)} \equiv 1 \ (mod \ m)$ 
- Raise to totient, under modulus of the prime/num passed to totient yields the neutral element. 
- The only condition needed is $gcd(a, m) = 1$; they are coprime. 
- This works for any integer ring $\mathbb{Z}_m$ 
	multiplication may not be commutative in a ring, and it almost always does not have all multiplicative inverses; if this were true then the integers would obviously constitute a field. 


### RSA key gen
1. Choose two large primes $p$ and $q$ 
2. Calculate the modulus $n = p \cdot q$ 
3. Compute the totient $\Phi(n) = (p-1)(q-1)$ 
4. Choose a value $e \in \{2, ..., \Phi(n)-1\}$ where $gcd(\Phi(n), e) = 1$ 
5. Compute $d$ where $d \cdot e \equiv 1 \ mod(\Phi(n))$ 

#### Example 
- $p = 17, q = 11$ 
	- In practice choose very large primes.
- $n = 187$ 
- $\Phi(n) = 160$, 160 nums between 1 and 186 that are co-prime with 187
- $e = 3$, $gcd(160, 3) = 1$ 
- $107 \cdot 3 \equiv 1 \ mod ( 160 )$ 

The public key is $n$ and $e$. The private key is $p, q, \Phi(n)$ and $d$, but we just take $d$.

COMPUTE D via EEA


### Breaking RSA
- To break RSA, must compute the totient to yield the modulus to uncover $d$ 
- However, $p$ and $q$ are kept private; one would have to factor all primes with regards to $n$ and uncover a combination that worked for the rest of the algorithm.
	- Would have to compute the totient and factor $n$ for all primes, which for large primes is essentially impossible. 


### RSA - Encryption
- Now that we have a public key $(3, 187)$ and a private key $107$ 
- Encryption and decryption is performed by:
	$x^e \equiv y \ (mod \ n)$ 
	$y^d \equiv x \ (mod \ n)$ 
- For example:
	- x = 74
	- x^3 = 74 ^ 3 (182) mod 187 = y
	- y^107 = 182^107 mod 187 = x
- The proof essentially boils down to the commutativity of integer exponentiation within rings, whilst under a modulus, and euler's totient applies.


### Why RSA is secure
- The attacker has public key $(n, e)$ and is trying to uncover the private key $d$ to uncover the plaintext from $y$
- In order to do so, they need to solve:
	- $y^?  \equiv x \ mod(n)$ 
- Computing $d$ requires solving $d \cdot e \equiv 1 \ (mod \Phi(n))$ , which requires knowing the totient of $n$
- This means one needs to know $p$ and $q$ to determine the totient for $n$
- For a 2048 but $n$ computing $p$ and $q$ factors by brute force and trying the result is computationally infeasible, thus, one cannot recover $d$ and thus the plaintext. 


### Exponentiation
- RSA requires computing things like $x^e$ 
- Naive approach, multiply x by itself e times, works where $\Phi(n)$ is small
	- This works where $\Phi(n)$ is small, thus the integer set we choose $e$ from is also small
- This is impractical as in practice exponents can be thousands of bits long
- For example
	$x^8$ requires 7 multiplications
	x.x = x^2
	x^2 . x  = x^3 
	...
- We can shorten by repeatedly squaring
	- $x \cdot x = x^2$ 
	- $x^2 \cdot x^2 = x^4$
	- $x^4 \cdot x^4 = x^8$ 

### Square and multiply
- Introduce multiplications rather than just squaring too
- Target exponent is represented in binary
	![](Pasted%20image%2020260309193551.png)
- Carry out square and multiply operations such that it matches the exponent
- $![](Pasted%20image%2020260309193703.png)
- Square = left shift
- Multiplication = add 1
DONE ONTO THE XS
### Comp complexity of large bit exponentiatiton
- 1.5T operations in theory; for every 0 have to do a square, for every 1 have to do a multiply. 


### 65537
- The largest known number of the form $2^{2^n}+1$ 
- Primes of this form = fermat primes
- Takes the representation $2^{2^4}$ = $2^16 + 1$ 
- This makes its binary representation full of zeroes, which means there are just 16 squarings, and 2 multiplies, making just 18 operations to encrypt, and so is the standard public e value. 