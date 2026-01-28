## Historical ciphers
- Ciphers have been used for thousands of years
- Most early ciphers are based on:
	- Substitution(replace symbols)
	- Transposition(rearrange symbols)

### Caesar & Shift ciphers
- Caesar cipher replaces each letter with another a fixed number of places $k$ down the alphabet
- Caesar always used $k=3$ 
- The caesar cipher is a **special case of the shift cipher**
- We can express it mathematically via **modular arithmetic**

### Modular arithmetic
- A system of arithmetic for **finite sets of integers**
	- Common infinite sets e.g., natural numbers, integers, complex numbers, real numbers etc.
	- Cryptography almost always interested in finite sets
- Essential for cryptography
- A clock(mod 12) is the standard example; wrapping around after reaching the modulus. 

#### Congruence
- We write:
	- $a \equiv r \ (mod \ m)$ 
- Meaning:
	- $m \ ∣ \ (a-r)$ 
	- $m$ divides $(a-r)$ 
- Where $a, r, m \in \mathbb{Z}$ and $m >0$  
- This means that with respect to a modulus, $a$ and $r$ are the same (their difference is a multiple of $m$), meaning they are congruent. 
	![](Pasted%20image%2020260127200024.png)
- The congruence relation:
	- $a \equiv r \ (mod \ m)$ 
- Can be written as:
	- $a = q \cdot m + r$ 
	 ![](Pasted%20image%2020260127200200.png)'
	- The remainder is not unique: rewriting like this gives us ways to form $a$ given a modulus $m$ and choosing a remainder $r$, to denote that $a$ and $r$ are congruent.

### Equivalence classes
- Integers can be grouped into **equivalence classes** as ordained by modulo $m$
- Example (mod 5):
	- All integers belong to one of five classes: {0}, {1}, {2}, {3}, {4}
	- Each class represents infinitely many integers.
![](Pasted%20image%2020260127200536.png)

### Integer Rings
- Modular arithmetic forms in Abstract Algebra what would be referred to as a **ring**:
	***Definition***:
	The integer ring $\mathbb{Z}_m$ consists of:
	1. The set $\mathbb{Z}_m = \{0, 1, ..., m-1\}$
	2. Two operations $+$ and $\cdot$ for all $a, b \in \mathbb{Z}_m$ such that:
		$a)$ $a+b \equiv c \ (mod \ m), (c \in \mathbb{Z}_m)$ 
		$b)$ $a \cdot b \equiv d \ (mod \ m), (d \in \mathbb{Z}_m)$ 
- We can add or multiply any two numbers in the ring, and the result is in the ring. It is closed under addition and multiplication. 
- Addition and multiplication are **associative.**
- There is a **neutral element** $0$ for addition.
- The additive inverse for $a$ always exists.
- There is a **neutral element** $1$ for multiplication
- The multiplicative inverse e.g., $a \cdot a^{-1} \equiv 1 \ mod \ m$ exists for some, but not all elements

#### Modular Inversion
- Multiplicative inverses allow us to 'divide' by a number
- Not all numbers in a ring have an inverse; you can determine whether one exists quite simply, if:$$gcd(a, m) = 1$$Then an inverse exists, where $a \cdot x \equiv 1 \ (mod \ m)$ 
- Example:
	$3 \cdot 9 \equiv 1 \ (mod \ 26)$ 
	$9$ is the modular inverse of $3$ mod $26$.

### Shift Cipher
- We can formalise the shift cipher using modular arithmetic:
	- Let $x, y, k \in \mathbb{Z}_{26}$ 
		- $e_k(x) = y \equiv x + k \ (mod \ 26)$ - Encrypt
		- $d_k(y) = x \equiv y - k \ (mod \ 26)$ - Decrypt

### Frequency analysis
> A method of breaking substitution ciphers by exploring **letter frequency patterns.**

- The longer the ciphertext, the easier this becomes
- Some letters appear more than others
- Can match high-frequency letters to common plaintext letters to gradually reconstruct the key.

### Affine Ciphers
- We can extend the shift cipher into an **affine cipher** by equipping it with multiplication and addition, rather than just shifting.
	- Let $x, y, a, b \in \mathbb{Z}_{26}$ 
		- $e_k(x) = y \equiv a \cdot x + b \ (mod \ 26)$ 
		- $d_k(y) = x \equiv a^{-1} \cdot (y - b) \ (mod \ 26)$ 
	- ![](Pasted%20image%2020260127213351.png)

![](Pasted%20image%2020260127213403.png)
