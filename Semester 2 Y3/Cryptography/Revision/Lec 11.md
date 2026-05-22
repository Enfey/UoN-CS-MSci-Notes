## Diffie Hellman
- Two parties can derive same shared secret computing secret independently whilst only transmitting public values
- Eavesdropper cannot reconstruct the secret.


### Groups
> A groups is a set of elements $G$ equipped with an operation $\circ$ such that:

1. $\forall a,b \in G, a \circ b = c \in G$ 
2. $\forall a, b, c \in G, a \circ (b \circ c) = (a \circ b) \circ c$ 
3. There is a neutral element such that $a \circ n = n \circ a = a \in G$ 
4. $\forall a \in G$ there exists $a^{-1} \in G$ called the inverse of $a$ such that $a \circ a^{-1} = a^{-1} \circ a = n$ 
5. Additionally, a group is **abelian** if the operation is commutative. 

### $\mathbb{Z}^*_n$ 
- The group $\mathbb{Z}^*_n$ consists of the integers $\{0, 1, 2, ... n-1\}$ for which $gcd(i,n) \equiv 1 \ (mod \ n)$ 
	- The set of integers that are co-prime to $n$ form an abelian multiplicative group(they now have inverses by gcd)
- The identity element is 1
- The often use a prime number as the modulus by which the operation is governed, similar to prior. 
- Ensures group is closed under multiplication, and ensures all group elements are co-prime to $p$:$$\mathbb{Z}_{p}^* = \{1, 2, \dots, p-1\}$$

### Group Cardinality
- The cardinality of a group is the number of elements in that group
	- $\vert \mathbb{Z}^*_p \vert$ = $p-1$ 
	- $\vert \mathbb{Z}_n^* \vert$ = $\Phi(n)$ 
		- Yields the number of integers co-prime with $n$ which is precisely the membership condition for this group, as without it, not all elements would have a multiplicative inverse.
			- The modulus is performed by $n$ you see. 

### Cyclic groups
- Let us now look at $\mathbb{Z}_{11}^*$ 
- Consider calculating the powers of 3 in this group
	- $3^i  \ mod \ 11$ 
		- 3^ 1 = 3 mod 11
		- 3^ 2 = 9 mod 11
		- 3^3 = 5 mod 11 etc.

### Order of an element of a group
- The order $ord(a)$ of an element of the group is the smallest positive integer $k$ such that $a^k = a \circ a \circ a \circ \dots \circ a \circ a = 1$ 
	- Where 1 is the neutral element
- We ask how many times do you apply the group operation $\circ$ to obtain the neutral element/identity?

#### Example
 - $2^i$, $\mathbb{Z}^*_{11}$ 
	 2^1 = 2 mod 11
	 2^2 = 4 mod 11
	 ...
	 2^10 = 1 mod 11
- Powers of two actually hit, every single group/equivalnce class n the group before cycling back to the neutral element
- This means that the $ord(2) = k = 10 = \vert \mathbb{Z}^*_{11} \vert = (p-1)$
- This means the order of this element is equivalent to the **group cardinality**, and thus has maximum possible order.
MAXIMAL ORDER MEANS EQUIV TO THE GROUP CARDINALITY



### Cyclic Group definition
- Any group that contains an element $g$ which has **maximum order** (same as the group cardinality $\Phi(n)$ ) is called a **cyclic group**
- This element is called a **generator**
- Thus, 2 is a generator of $\mathbb{Z}_{11}^*$ whilst 3 is not as its order $ord(3) = 5$ 
### Cyclic Subgroups
- For all primes, the group $\mathbb{Z}^*_p$ is:
	- **Abelian**
	- **Finite**
	- **Cyclic**
		- It can be proven that always contain at least one generator, usually multiple.
- Let $a \in G$ where $G$ is a cyclic group
	1. $a^{\vert G \vert} = 1$ 
		- Cannot arrive at the identity beforehand
	2. $ord(a)$ divides $\vert G \vert$ 
		- The order of **ANY ELEMENT** divides the group cardinality.
- For $Z^*_p$ we have multiple **cyclic subgroups**
- Each element raised to a power generates a new **cyclic subgroup** that corresponds directly to that elements maximal order.
	- The orders of these elements correspond to a new group:
		![](Pasted%20image%2020260309213540.png)
		![](Pasted%20image%2020260309213548.png)
	- For example group element 10 generates 1, 10, under mod 11, and as such the order 2 has the cyclic subgroup 1, 10
	- The number of distinct cyclic subgroups corresponds to the number of distinct orders in the cyclic group
	- The largest order's subgroup is equivalent to the entire group.
		- This is important for diffie-hellman; as choosing a generator with maximal order generates the entire outer cyclic group. 
		- ONLY ELEMENTS WITH MAXIMAL ORDER GENERATE THE ENTIRE CYCLIC SUBGROUP.
### Diffie-Hellman
1.  Alice and Bob publicly agree on a large prime $p$ and a generator $g$ with respect to $p$ meaning it generates the full group $\mathbb{Z}_p^*$ 
2. Alice and bob choose secret numbers $a$ and $b$ at random from the group
	$a \in {1, 2, \dots, p-1}$ $b \in {1, 2, \dots, p-1}$
3. Each raises $g$ to their private secret, and sends the result publicly
	- Alice computes $g^a \ mod \ p = A$ 
	- Bob computes $g^b \ mod \ p = B$ 
- Each takes each others public value, $A$ and $B$ and raises it to their own private secret
	Alice computes $B^a \ mod \ p = k_{ab}$
	Bob computes $A^b \ mod \ p = k_{ba}$ 
#### Proof
- Very straightforward
	$B^a = (g^b)^a = g^{ab} = (g^a)^b = A^b$
	Law of exponents.


### Why groups matter for DH
- **Closure**
	- When computing $A$ and $B$ the group operation applied repeatedly ensures that the element will remain in the group.
- **Cyclic structure**
	- Because $g$ is a generator, $g^a$ hits every possible element of the outer group, without skipping any, as $a$ varies
	- This means that the public values $A$ and $B$ are spread uniformly across the entire group, giving no information about $a$ or $b$ to an attacker; each element is equally likely.
	- IFF one used an element without maximal order and had a weak subgroup, an attacker could potentially brute force the order of that element and discover its subgroup and try its values. 

#### DLP
 - An attacker watching network sees:
	 $g = 3$ 
	 $p = 10000079$ 
	 Alice calculates $3^a \ mod \ 10000079 = 4675534$ 
- To find $a$ they'd need to invert the exponentiation; it is the only thing not transmitted. Finding $a$ from $g^a$ where $g$ is a generator has no efficient algorithm for large primes
- Only need either $a$ or $b$ to compute the shared secret, obviously, as then just apply to one of the public values

##### DLP hardness
- **Brute force**
	- $O(\vert G \vert)$ 
	- Muust try every exponent from 1 to p-1
- **Shank-s Baby step giant step**
	- Split the exponent $a$ into two halves, and search each half separately. 
	- $g^{mx+c}$ one precomputes all of mx and then solves for the constant, but this requires space with respect to the square root of the group cardinality. its time is also the square root of the group cardinality
- **Pollard's Rho**
	- Time: square root of the group cardinality, but does not require the space as it does not store a list
- Index calculus
	- Really effective for discrete logarithms, and is the **only reason** that elliptic curve groups are preferred (their elements are not integers)
	- Edxploits structure unique to integers and attacks $Z_p^*$ directly. 


#### Which prime?