## Recap of Elliptic Curves
- Elliptic curves provide a cyclic group that we can use to build public key cryptography
- An EC group is defined as follows:
	- Elements are the points on a curve that solve the equation $y^2 \equiv x^3 + ax + b \ (mod \ p)$ 
		- Arithmetic is governed by the outer prime field.
	- A group operation, namely **point addition** $P+Q = R$ which is associative, closed, has inverses via the $y^2$ portion of the defining equation, and commutes, under reflection. 
- We typically use curves over the field $\mathbb{Z}_p$ rather than $\mathbb{R}$
	- Doing computation with real numbers in implementations may forego some determinism, amongst other mathematical issues.
	- The field defines our modular addition and multiplication, as seen in the point addition equation; the field arises naturally.

## Cyclic Groups in Elliptic Curves
- Given a generator point $P$, we can generate cyclic EC groups.
- $E: y^2 \equiv x^3 + 2x + 2 \ (mod \ 17)$
	- Take $P = (5, 1)$ 
	![](Pasted%20image%2020260322220056.png)
	- $18P = (5, 16)$
		- The $x$ co-ordinate is the same; we know it is impossible to have an EC group element with the same $x$ co-ordinate that is not a corresponding inverse
		- The $18P + P = 19P$ would yield a divide by $0$ via the definition of the slope as the $x$ co-ordinate is equal.
			![](Pasted%20image%2020260322201335.png)
		- We determine whether the $y$ co-ordinate's are additive inverses of one another (1 and 16), which they are:
			- $1 + 16 \equiv 0 \ mod \ 17$, naturally this commutes.
			- We choose to write $18P = (5, 16) = (5, -1)$ to signal that $16$ is the additive inverse of $1$, written $-1$.
		- Thus, 19P yields the point at infinity definitionally: $19P = 18P + P = -P + P = \mathcal{O}$ 
		- $20P = P + \mathcal {O} = P$ naturally regenerating the prior group elements. So the order of $P$ is $19$.
	- From this we can see that any number of $P$ which adds up to $19P$ via point addition is also going to be at the point at infinity.
		- Say $10P$ and $9P$ added via point addition, yield $19P$.
		- Those elements will actually be additive inverses of one another, solving the requirement that $\mathcal{O}$ imposes.
		![](Pasted%20image%2020260322223543.png)

## ECDLP
- We can construct the DLP in a very similar manner to the modular exponentiation variant: $$
aP = \underbrace{P + P + \dots + P}_{a\ \text{times}} = A
$$
- Given points $P$ and $A$, find the scalar value $a$ that yielded $aP = A$. Solving this, for DH, amounts to breaking the cryptographic security of the key exchange, as we will see soon. 
- This is easy to compute forward, but computationally infeasible to reverse, and due to some unique characteristics of EC groups, is relatively immune to **index calculus** attacks.

## Points vs Integers
- it is important to note the distinction between points on the curve that are elements of the EC group, and integers that belong to the above prime field. 
- In Elliptic curve cryptosystems, private keys such as $a$ are integers
- Generators $G$ and public keys, such as $A$ and $B$ on the other hand, are points $(x, y)$ 

## Group Cardinality
- The size of cyclic groups is extremely important for security
	- Conceptually, if we had a giant elliptic curve with many elements, and we chose a generator which yielded a small cyclic sub-group and we were trying to solve DH, it would be easy to bruteforce the number of points on the curve.
- While this is easy to calculate for modular arithmetic (Euler totient for arbitrary modulo groups, and $p-1$ for prime groups), the number of points on a given Elliptic curve is not so obvious.
- One may imagine that a curve would have $2p + 1$ points (number of points defined with unique $x$ (coming from $p$) and their respective inverses with additive inverse $y$ + the neutral element, the point at infinity). This is not the case.
	- This intuition comes from naively assuming that every $x$ yields two points, which is not true.
	- Some $x$ values yield no solution whatsoever, we fix $x$ values in the elliptic curve equation and ask whether it solves $y^2 = k$, so we ask whether $k$ has a square root, which is predicated on the value of $x$ via $y^2 = x^3 + ax +b$
	- Over a large prime field, exactly half of all non-zero elements have square roots. If we have a valid $x$, then it has 2 points definitionally via $(x, y)$ and $(x, -y)$, and so on average, 50% of the $x's$, of which there are $p$ values, contribute 2 elements, yielding $p+1$ as our baseline. 
- **Hasse's Theorem** states that for a curve $E$ over a field $\mathbb{Z}_p$, the cardinality of the $EC$ group is bounded by:
	- $\#E = p + 1 + \epsilon$
		where $|\epsilon| \le 2 \sqrt p$ 
		Will usually not be exactly $p+1$, and this value is extremely neglible for large $p$.
		![](Pasted%20image%2020260322234853.png)

## \#E 
- Calculating the exact $\#E$ is difficult, but it can be done with **Shoof's algorithm**
- Various properties of $\#E$ enable or restrict certain attacks on the cyclic EC group
- A large $\#E$ is obviously very important for the group security, as a group element with maximal order will cycle through that cardinality, and make brute force attacks more difficult
	- Large $\#E$ is directly tied to $p$ then, so large primes make solving ECDLP very difficult
- It is recommended that we do not generate our own elliptic curves. The vast majority of applications use standard curves.

## How hard is ECDLP?
- Two algorithms apply to any group DLP regardless of structure
	1. **Pollard's Rho**
	2. **Pohlig-Hellman**
	Typically generic algorithms on the group are $O(\sqrt{\#E})$ 
- These generic attacks mean that curves and parameters should be chosen with care to make them infeasible; ensure large $\#E$ 
- Index calculus fundamentally does not work on elliptic curves
	- I don't know why, and it isn't covered.
	- Modern modular arithmetic cryptosystems use $>2000$ bit keys to be resistant to index calculus attacks, whereas ECDH only needs 256-bit keys.

## Efficient computation of $aP$ 
- There is no natural way of calculating $aP$
- We do not literally add $P$ to itself $a$ times, we use **double and add**:
	- We write $a$ in binary, e.g., if $a$ was $13$ then $1101_2$ 
	- When a $1$ appears, compute with respect to the binary
		- $13 = 1 \times 8 + 1 \times 4 + 0 \times 2 + 1 \times 1$ 
		- So $13P = 8P+4P+0P+1P$ 
		- Just need to compute these terms via doubling - 3 doublings - $P\to2P\to4P\to8P$ , and then 2 additions to combine them $8P+4P+P$
		- $5$ operations, which makes it feasible to go forwards.
- Or alternatively, as it appears in the lecture:
	![](Pasted%20image%2020260323011744.png)


## Elliptic Curve Diffie-Hellman
- $E$, $\#E$, $G$
- Alice
	- $a \in \{1, 2, \dots, \#E-1\}$ 
	- $A = a \cdot G$ 
	- $k_{ab} = a \cdot B$ 
- Bob
	- $b \in \{1, 2, ..., \#E-1\}$ 
	- $B = b \cdot G$ 
	- $k_{ba} = b \cdot A$ 
- Where $A$ and $B$ are exchanged.
### Proof of Correctness
- The proof is essentially the same as regular DH, utilising the associativity property of the cyclic group point addition operation, directly yielded as we remember, via its reflection characteristic.
	- We also use the natural commutativity of integer multiplication within the prime field $\mathbb{Z}_p$
- $k_{ab} = a \cdot B = a \cdot (b \cdot G) = ab \cdot G$ 
- $k_ba = b \cdot A = b \cdot (a \cdot G) = ba \cdot G = ab \cdot G$ 

### Example
![](Pasted%20image%2020260323013208.png)

### EC structure
![](Pasted%20image%2020260323013351.png)
This is yielded naturally. The scalar multiplication comes from e.g., double and add to perform multiplication of a point by a scalar value.
Changing the contents e.g., how to perform scalar multiplication, of one layer will not affect the things above, the inherent structure is what matters.
## Implementation
### Point Compression

### Projective Co-ordinates

### Standard Curves

### Duel_EC_DRBG