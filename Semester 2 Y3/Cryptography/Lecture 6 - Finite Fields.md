## Groups
> A group is a set of elements $G$ and an operation $\circ$ that combines two elements of $G$:

1. The operation $\circ$ is **closed**. $\forall a, b \in G$ then $a \circ b = c \in G$ 
2. The operation is associative. I.e., $a \circ (b \circ c) = (a \circ b) \circ c$ for all $a, b, c \in G$  
3. There is an element called a **neutral element/identity element** such that $a \circ 1 = 1 \circ a = a$ for all $a \in G$.
	This identity is usually either 1(addition), 0(multiplication), or $e$ (abstract).
4. For each $a \in G$ there exists an element $a^{-1} \in G$, called the **inverse** of $a$ such that $a \circ a^{-1} = a^{-1} \circ a = 1$ 
	Each element commutes with its own inverse, not with all elements of the group, unless it is an **abelian group**.
5. A group is **abelian** if the operation is commutative, for all $a, b \in G$, $a \circ b = b \circ a$.

### Example Group
- The set of integers $\mathbb{Z}_m = \{0, 1, ..., m-1\}$ with the operation addition modulo $m$ form a group with the neutral element $0$.
- Every element $a \in G$ would have a modular inverse $-a$ where $a + (-a) = 0 \ mod \ m$ 
	E.g., under mod $7$ addition, the inverse of $3$ is $4$
		$3 + 4 \equiv 7 \equiv 0 (mod \ 7)$ 
- Integers under $mod \ m$ multiplication do NOT form a group, because not every element has an inverse. 
	- Check modular multiplicative inverse, we need $a \cdot b \equiv 1 (mod \ m)$
		- This only happens when $gcd(a, m) = 1$
			![](Pasted%20image%2020260224200917.png)

## Rings

## Fields
> A field $F$ is a set of elements with the following properties:

1. All elements of $F$ form an **additive group** with the group operation $+$ and the neutral element $0$.
2. All elements of $F$ except 0 form a **multiplicative group** with the group operation $\times$ and the neutral element $1$. 
	0 has no multiplicative inverse.
3. When the two group operations are mixed, the **distributivity law** holds.
	For all $a, b, c \in F$, $a \cdot (b + c) = (a \cdot b) + (a \cdot c)$ 

Note that $0$ must be in the field as it is the additive neutral element, we could define a field $F$ as:
- An additive group and multiplicative group $(G, +) \cup (G \backslash \{0\}, \cdot)$ 
### Example Field
- The set of real numbers $\mathbb{R}$ is a field with a neutral element $0$ for addition, and $1$ for multiplication
- Every real number has an additive inverse $-a$ 
- Every non-zero number $a$ has a multiplicative inverse, $1/a$, to yield $1$.
- The complex numbers $\mathbb{C}$ form a field, the natural numbers $\mathbb{N}$ do not. 

In cryptography, we are almost always interested in **Finite Fields**.

### Finite Fields
> A **Finite Field** is a **Field** containing a finite number of elements.

- This is sometimes called a **Galois Field**, denoted GF($p^m$)
- Fields as seen prior, are an extension of **Groups**.

A finite field *only exists* if it has $p^m$ elements, where $p$ is a **prime number**, and $m$ is a **positive integer**.
- There is a field with $11$ elements $GF(11)$ 
- There is a field with $81$ elements $GF(81)$ or $GF(3^4)$ 
- There is a field with $256$ elements $GF(256)$ or $GF(2^8)$ 
- There is not a field with $12$ elements, no way to represent this via $p^m$.

### Prime and Extension Fields
- All prime and extension fields are types of finite fields
- A **prime field** is the simplest possible finite field, it has $p$ elements, with $m = 1$, where arithmetic is under $mod \ p$. 
- An **extension field** is a larger finite field built from a prime field which has $p^m$ elements, with $m \gt 1$ 
	- We call it this as it contains the prime subfield $GF(p)$ inside of it but uses the extension degree of $m$ to extend that subfield.
- Hugely important to cryptography are fields $GF(2^m)$ 
#### Prime Fields
- A prime field is the simplest possible finite field.
- It has exactly $p$ elements where $p$ is prime. The elements are just $\{0, 1, \dots, p-1\}$
- Arithmetic is done under $mod \ p$, let $a, b \in GF(p^1) = \{0, 1, \dots, p-1\}$ 
	- **Addition**: $a + b \equiv c \ (mod \ p)$ 
	- **Subtraction**: $a - b \equiv d \ (mod \ p)$
		- Automatically permitted in any field; it is simply addition + additive inverse; $a + (-b) = a-b$ 
	- **Multiplication**: $a \cdot b \equiv e \ (mod \ p)$ 
- These operations satisfy the properties of fields e.g., closure, distributivity when mixing, associativity etc. 

##### Inversion in Prime Fields
- All members of a prime field have a multiplicative inverse. 
- For a nonzero element $a \in F$, its inverse $a^{-1}$ is defined by:$$a \cdot a^{-1} \equiv 1 \ (mod \ p)$$
- From modular arithmetic, we can determine if $a^{-1}$ exists if $gcd(a, p) = 1$.
- For prime fields, every $a \in \{1, ..., p-1\}$ is not divisible by $p$, and $p$ is only divisible by itself and $1$ therefore $gcd(a, p) = 1$ for all $a \neq 0$. 
- We can calculate $a^{-1}$ for any element $a$ using the **Extended Euclidean Algorithm**.


#### Extension Fields
> An **extension field** is a larger finite field built from a prime field which has $p^m$ elements, with $m \gt 1$.

- In prime fields, the elements are integers. 
- Elements in extension fields $GF(2^m)$ are polynomials of degree less than $m$, with coefficients as elements of the prime sub-field. 
- They contain elements of the form: $$A(x) = a_{m-1}x^{m-1} + \dots + a_{1}x + a_{0}$$
	Where $a_i \in GF(2) = \{0, 1\}$ 
- Cryptography, important extension field $GF(2^m)$ as each coefficient is just a bit, and so each element represent an $m-bit$ sequence, and addition is simply XOR (addition under mod 2 for bits) because of $p = 2$.
##### Example of $GF(2^3)$ 
- The extension field $GF(2^3)$ sometimes called $GF(8)$ contains elements of the form $$A(x) = a_{2}x^2 + a_{1}x_{1} + a_{0}$$
- It is often easier to simply write the coefficients $(a_2, a_1, a_0)$ e.g., $001$ or $101$ for a given element (though this is obvious from looking at it.)
- The elements of $GF(2^3)$ are as follows:
	- $GF(2^3) = \{0, 1, x, x+1, x^2, x^2 + 1, x^2 + x, x^2+x+1\}$ = $8$ elements.

##### Arithmetic in $GF(2^3)$ 
- Adding or subtracting two polynomials occurs by adding the coefficients
- However this addition/subtraction happens under the prime-subfield which the coefficients are elements of (as that is how their operations are defined, cannot do arbitrarily). 
- For two polynomials:
	- $A(x) = x^2 + x + 1$ 
	- $B(x) = x^2      +1$ 
	- $A(x) + B(x)$
		$=(1+1)x^2 + x + (1+1)$
		$=(0)x^2 + x + (0)$ 
		= $x$


##### Multiplication in $GF(2^3)$
- $A(x) = x^2 + x + 1$
- $B(x) = x^2 + 1$ 
- $A(x) \cdot B(x)$
	$= (x^2 + x + 1) (x^2 + 1)$
	$=x^4 + x^2 + x^3 + x + x^2 + 1$ 
	$= x^4 + x^3 + (1 + 1) \cdot x^2 + x + 1$ 
	$= x^4 + x^3 + x + 1$ 
	The result is not in the field. 
- In a prime field, we do modular reduction by $p$, but we have a polynomial. We must reduce the intermediate result by an **irreducible polynomial** instead. 
- This is given and does not need to be calculated; much like $p$ is given for arithmetic in prime fields, this is given for multiplication in extension fields. 
- We know when to stop the polynomial long division and take the remainder when the valid elements have degree $< 3$ or degree $< m$
- $P(x) = x^3 + x + 1$ 
- $A(x) \cdot B(x) = x^4 + x^3 + x + 1 \ (mod \ x^3 + x + 1)$ 
	- Remember that all arithmetic is under $mod \ p$/*XOR*
		![](Pasted%20image%2020260224222343.png)

##### Inversion in Extension Fields
- Very similar to prime fields, we perform inversion like so:
	- $A(x) \cdot A^{-1}(x) \equiv 1 \ (mod \ (P(x))$ 
- As with prime fields, we can calculate $A^{-1}(x)$ using the Extended Euclidean Algorithm. 
#### AES' Finite Field
- AES uses the extension field GF(2^8), so up to degree 7 for our polynomials.
- Operations are the same as those in other GF(2^m), using the irreducible polynomial: $$P(x) = x^8 + x^4 + x^3 + x + 1$$
	Multiplication and inversion is governed by this. Arithmetic is governed by $XOR$ according to the prime subfield $GF(2)$.
- Polynomials are thus represented as single bytes. 

### Finite Field Arithmetic Worksheet (Revision)
