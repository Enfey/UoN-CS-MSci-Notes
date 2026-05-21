## Historical ciphers
- Used for thousands of years, 
	- Early ciphers, based on:
		- **Substitution** - replace symbols entirely
		- **Transposition** - rearrange symbols
## Caesar & Shift ciphers
- Caesar cipher replaces each letter with another, a fixed number of places down alphabet
- Caesar always used $k = 3$ and as such is a instantiation of the **shift cipher**
- We can express this mathematically via modular arithmetic. 

## Modular arithmetic
- System of arithmetic for finite sets of integers
- Cryptography almost always interested in finite sets

### Congruence
- We write $a \equiv r \ (mod \ m)$ to denote that $a$ and $r$ are **congruent** under modulo $m$
	- This means that $m \ \vert \ (a - r)$; $m$ divides their remainder.
	- Their difference is a multiple of $m$
- Where $a, r, m \in \mathbb{Z}$ and $m > 0$ 
- The congruence relation can be written as:
	- $a = q \cdot m + r$ 
		A number of $m$'s and a remainder formulate $a$
	- The remainder is not unique; rewriting like this gives us ways to form $a$ given a modulus $m$ and $a$, and choosing a remainder $r$ to denote the remainder(s) that $a$ is congruent with under that modulo.


### Equivalence classes
- Integers can be grouped into equivalence classes as ordained by the modulo $m$
- E.g., under $mod \ 5$ all integers belong to one of 5 classes, or implicit sets, {0}, {1}, {2}, {3}, {4} where each class represents infinitely many integers.


### Integer rings
- Modular arithmetic forms in Abstract Algebra what would be referred to as a ring:
The integer ring $Z_m$ consists of:
1. The set $\mathbb{Z}_m = \{0,1, ..., m-1\}$ 
	Remainders modulo $m$ that every integer collapses to one of these equivalence classes in the ring
2. Two operations $+$ and $\cdot$ for all $a, b \in \mathbb Z_m$ such that:
	$a+b \equiv c \ (mod \ m), (c \in \mathbb{Z}_m)$ 
	$a \cdot b \equiv c \ (mod \ m), (d \in \mathbb{Z}_m)$
- We can add or multiply any two numbers in the ring, and the result is in the ring; it is thus closed under addition and multiplication
- Neutral element is zero for addition, and 1 for multiplication
- Additive inverse for $a$ always exists
- The multiplicative inverse $a \cdot a^-1 \equiv 0 \ (mod \ m)$ exists for some, but not all elements. 
- Addition and multiplication are **associative**


### Modular inversion
- 