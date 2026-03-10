## Diffie-Hellman
- Two parties can derive the same shared secret, computing the secrete independently without ever explicitly transmitting it. 
- Everything sent over the network is public, yet an eavesdropped cannot reconstruct the secret.

## Groups
- A group is a set of elements $G$ together with an operation $\circ$ that combines two elements of $G$:
	1. The operation $\circ$ is closed, $\forall a, b \in G, a \circ b = c \in G$ 
	2. The operation is associative, $\forall a, b, c \in G, (a \circ b) \circ c = a \circ (b \circ c)$ 
	3. There is a neutral element such that $a \circ n = n \circ a = a \in G$ 
	4. For each $a \in G$ there exists an element $a^{-1} \in G$ called the inverse of $a$ such that $a \circ a^{-1} = n$, and each $a$ commutes with its own inverse. 
	5. Additionally, a group $G$ is **Abelian** if $\circ$ is commutative, $\forall a, b, a \circ b = b \circ a$ 
Closed, associative, neutral element, inverse, commutativity of $\circ$. 
### Example Groups
- The integers in $(\mathbb{Z}, +)$ form an abelian group.
- What about $\mathbb{Z_9} = \{0, 1, 2, 3, 4, 5, 6, 7, 8\}$ and the multiplication operation? (under $mod \ 9$)
	- No multiplicative inverse ($gcd(6, 9), gcd(3, 9) \neq 1$, 0 has no multiplicative inverse in any group that it belongs to.)
- We remove the elements which are problematic and do not allow us to form a full group, they are 0, 3 and 6 as they have no multiplicative inverses. 
- Our new group is defined as $\mathbb{Z}^*_9 = \{1, 2, 4, 5, 7, 8\}$ 

## $\mathbb{Z}^*_n$ 
- The group $\mathbb{Z}^*_n$ consists of the integers $\{1, 2, \dots, n - 1\}$ for which $gcd(i, n) = 1$ where $i \in \mathbb{Z}^*_n$. 
	- That is the set of integers that are co-prime to $n$, form an abelian group under multiplication $mod \ n$.
- The identity element, naturally, is one. 
- In the majority of cases, we use a prime number as the modulus by which the operation $\circ$ is governed:$$\mathbb{Z}^*_{p} = \{1, 2, \dots, p-1\}$$
- This is because every number below a prime is automatically coprime to it. 

## Group Cardinality
- The cardinality of a group is the number of elements in that group:
	- $|\mathbb{Z}^*_p| = p-1$ 
	- $|Z^*_n| = \Phi(n)$ 
		- Yields the number of integers co-prime with $n$ in terms of $n's$ factors, as discussed in the prior lecture. 
		- This is of course, precisely the membership condition for $\mathbb{Z}^*_n$. 


## Cyclic Groups
- Let us now look at the group $\mathbb{Z}^*_{11}$ 
- Consider calculating powers of $3$ in this group:
	- $3^i \ mod \ 11$ 
		- $3^1 = 3 (mod \ 11)$ 
		- $3^3 = 5(mod \ 11)$
		etc

### Order of an element of a group
- The order $ord(a)$ of an element $a \in G$, $(G, \circ)$ is the smallest positive integer $k$ such that: 
	- $a^k = a \circ a \circ a \circ \dots \circ a = 1$ 
	- Where $1$ is the neutral element of $G$. 
- We ask: how many times do we apply the group operation before you cycle back to the identity?


#### Example for Cyclic Groups
- What about $2^i$ in $\mathbb{Z}^*_{11}$?
- $2^i \ mod \ 11$ 
	- ![](Pasted%20image%2020260309204520.png)
	- Powers of $2$ hit every single element in the group before cycling back to the neutral element. 
	- This means that $ord(2) = k = 10 = |\mathbb{Z}^*_p| = p-1$, the maximum possible order, as this is **equivalent to the group cardinality**
### Cyclic Group definition
- Any group that contains an element $g$ which has maximum order is called a cyclic group
- This element is called a **generator**.
- For example, 2 is a generator of $\mathbb{Z}^*_{11}$, whilst $3$ is not, as $ord(3) = 5$. 


#### Cyclic Subgroups
- For all primes, the group $\mathbb{Z}^*_p, \cdot$ is: (we use multiplication for DH, see later on why)
	- **Abelian**
	- **Finite**
	- **Cyclic**
		- It can be proven that always contain at least one generator/primitive root (usually multiple).
- Let $a \in G$ where $G$ is a cyclic group:
	1. $a^{|G|} = 1$  
		- May already have arrived at the neutral element prior, but will arrive at the neutral element $1$ at this exact point also.
	2. $ord(a)$ divides $|G|$ 
		- The order of any element is a divisor of the group's cardinality.
- For $\mathbb{Z}^*_p$ ,  we have multiple **cyclic subgroups**
- Each element raised to the power of an order generates a cyclic subgroup that corresponds directly to that order.
	- For example:
		![](Pasted%20image%2020260309213540.png)
		![](Pasted%20image%2020260309213548.png)
		Element $10$ generates $\{1, 10\}$ under $mod$ $11$, so the order 2 has the cyclic subgroup $\{1, 10\}$ 
	- The number of distinct cyclic subgroups corresponds directly to the orders/factors of $\mathbb{Z}^*_p$.
	- The largest order's subgroup is equivalent to the entire group. This is important for diffie hellman, as choosing a generator with this order generates the entire group as its cyclic subgroup when used as an exponent for $a$. 


### Diffie Hellman
1. Alice and Bob publicly agree on a large prime $p$, and a generator $g$ with respect to $p$, meaning it generates the full group $\mathbb{Z}^*_p$ 
2. Alice and Bob choose private secret numbers $a$ and $b$ at random in $\mathbb{Z}^*_p$ 
	![](Pasted%20image%2020260309214426.png)
3. Each raises $g$ to their private secret, and sends the result publicly
	- Alice computes $A = g^a \ mod \ p$ and sends $A$ to Bob. 
	- Bob compute $B = g^b \ mod \ p$ and sends it to Alice.
	![](Pasted%20image%2020260309214752.png)
4. Each takes the others public value, $A$ and $B$ and raises it to their own private secret.
	- Alice computes $k_{ab} = B^a \ mod \ p$
	- Bob computes $k_ab = A^b \ mod \ p$ 

#### Proof of Correctness
- Very straightforward.
	$B^a = (g^b)^a = g^{ab} = (g^a)^b = A^b$
	The middle value arises as a standard rule of exponents. 
		For bob, $(g^b)^a = g^{(b \cdot a )} = g^{ab}$
	The middle value also arises because multiplication is commutative. 


### Why groups matter for DH
- **Closure**
	- When computing $A$ and $B$, the result is guaranteed to stay inside the group.
- **Cyclic structure**
	- Because $g$ is a generator, $g^a$ hits every possible element of $Z^*_p$ without skipping any, as $a$ varies. 
	- This means that the public values $A$ and $B$ are spread uniformly across the entire group, giving no information about $a$ or $b$ to an attacker.
	- If we used a $g$ without maximum order, would yield a small subgroup and skip values, making a brute force trivial. 

### Example Diffie Hellman
![](Pasted%20image%2020260309220458.png)


### The Discrete Logarithm Problem
- An attacker watching the network sees, for the following example:
	- $g = 3$
	- $p = 10000079$ 
	- Alice calculates = $A = 3^a \equiv 4675534 \ (mod \ 10000079)$
- To find $a$ they'd need to invert the exponentiation. Finding $a$ from $g^a$ has no efficient algorithm for large primes. 
- Only need either $a$ or $b$ to compute the shared secret.

#### How hard is the discrete logarithm problem
- **Brute-Force**
	- Try every possible exponent from $1$ to $p-1$ until find where $g^a ≡ A (mod \ p)$
	- $O(|G|)$
- **Shank's Baby-Step Giant-Step**
	- Split the exponent $a$ into two halves, and search each half separately. 
	- $O(√|G|)$ and requires roughly $√|G|$ space.
	- Instead of doing $g^x$, do $g^{mx + c}$ 
		- Precompute all $mx$, then can solve for the constant?
- **Pollard's Rho**
- **Pohlig-Hellman**
- **Index Calculus** really effective for discrete logarithms, and is the reason that Elliptic Curve groups are so much more efficient. 


### Which prime?
- To avoid any unexpected small subgroup attacks, commonly used DH primes are **safe primes**
- A safe prime is a prime $p$ where $(p-1)/2$ is also a prime. 
	- Generators generate cyclic subgroups equivalent to the size of the group, where the size of the group is $p-1$ 
- Consider the order of $\mathbb{Z}^*_p$ '
- 
- 
- 
- COVER THIS LATER'

