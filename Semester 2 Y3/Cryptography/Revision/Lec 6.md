## Groups
> A group is a set of elements $G$ and an operation $\circ$ that combines 2 elements of $G$:
1. The operation $\circ$ is **closed**. $\forall a, b \in G, a \circ b = c \in G$ 
2. The operation is **associative** I.e., $a \circ (b \circ c) = (a \circ b) \circ c, \forall a, b, c \in G$ 
3. There is a **neutral element**/identity element such that $a \circ 1 = 1 \circ a = a$, $\forall a \in G$ 
	Usually 1 for mult, 0 for addition, e for abstract
4. For each $a \in G$ there exists an element $a^{-1} \in G$ such that $a \circ a^-1 = 1$ 
	Each element commutes with its own inverse, not with all elements of the group unless it is **abelian**
5. A group is **abelian** if the operation $\circ$ is commutative $\forall a, b \in G$, $a \circ b = b \circ a$ 


### Fields
> A field $F$ is a set of elements with the following properties:

1. All elements of $F$ form an **additive abelian group** with the group operation $+$ and the neutral element $0$
2. All elements of $F$ for a **multiplicative abelian group** with the group operation $\times$ and the neutral element is $1$
	- The element $0$ need not have a multiplicative inverse; it is in the group regardless because of the presence of the additive abelian group
3. When the two group operations are mixed (multiplication over addition) the distributivity law holds:
	$a, b, c \in F a \times (b + c) = (a \times b) + (a \times c)$ 

Note that $0$ must be in the field as it is the additive neutral element, we could define a field $F$ as:
- An additive group and multiplicative group $(G, +) \cup (G \backslash \{0\}, \cdot)$ 


### Finite Fields
> A **Finite Field** is a **Field** containing a finite number of elements

- This is sometimes called a **Galois Field** denoted $GF(p^m)$
- Fields as seen prior, are an extension of **Groups**

A finite field *only exists* if it has $p^m$ elements where $p$ is a **prime number** and $m$ is a **positive integer** 
	There is a field with 11 elements $GF(11^1)$ 
	There is a field with 81 elements $GF(3^4)$ 
If you can express it via $p^m$, then it exists as a finite field.


### Prime and Extension fields
- All prime and extension fields are variants of finite fields.

#### Prime fields
- A **prime field** is the simplest possible finite field; it has $p$ elements, with $m=1$ where the arithmetic is under $mod \ p$ 
- It has exactly $p$ elements where $p$ is prime, where the elements are $\{0,1,2,\dots, p-1\}$ 
- Arithmetic is done under $mod \ p$; let $a, b \in GF(p^1) = \{0, 1, \dots, p-1\}$ 
	- **Addition**: $a + b \equiv c \ (mod \ p)$ 
	- **Subtraction**: $a-b \equiv d \ (mod \ p)$ 
		- Permitted; simply addition + additive inverse
			 $a + (-b) = a-b$ 
	- **Multiplication**: $a \times b = c \ (mod \ p)$ 
- These operations of course, satisfy associativity, distributivity, commutativty, closure etc. 



##### Inversion in prime fields
- All members of a prime field have a multiplicative (and additive) inverse
- For a nonzero element $a \in F$ its inverse $a^{-1}$ exists if $gcd(a, p) = 1$ 
- For prime fields every $a \in \{1, 2, \dots, p-1\}$ is not divisible by $p$ and $p$ is only divisible by itself and $1$ therefore $gcd(a, p) = 1$ for all $a \neq 0$ 
- Calculate $a^{-1}$ via EEA

#### Extension fields
> An **extension field** is a larger finite field built from a prime field which has $p^m$ elements with $m > 1$ 

- In prime fields, the elements are **integers**
- Elements in extension fields $GF(2^m)$ are polynomials of degree less than $m$ with coefficients as elements of the prime sub-field. 
- They contain elements of the form:$$A = a_{m-1}x^{m-1} + \dots + a_{1}x + a_{0}$$ 
	Where $a_i \in GF(2) = \{0, 1\}$ 
- An important extension field in cryptography is $GF(2^m)$ where the polynomial denotes an $m$ bit sequence and if $a$ is on then yes, the bit is on
	- What x is doesn't really matter
- For example the elements of $GF(2^3)$ are as follows:
	- $\{0, 1, x, x+1,x^2, x^2 + x, x^2 + 1, x^2 + x + 1\}$ 
	- Such that $A(x) = a_2x^2 + a_{1}x+a_{0}$ 

#### Arithmetic under prime extension fields
- Adding or subtracting two coefficients occurs by the two coefficients $a_i$ as the addtion is governed by the prime-sub field $GF(2)$ and as such, occurs under that modulus.
	$A(x) = x^2 + x + 1$ 
	$B(x) = x^2 + 1$ 
	$= (1+1)x^2 + x + (1+1)$ 
	$= x$  

#### Multiplication under prime extension fields
- $A(x) = x^2 + x + 1$ 
- $B(x) = x^2 + 1$ 
	=$(x^2 + x + 1)(x^2 + 1)$ 
	$=x^4 + x^3 + x^2 + x^2 + x + 1$
	$=x^4 + x^3 + (1+1) \cdot x^2 + x + 1$ 
	$= x^4 + x^3 + x + 1$ 
	The result, is not in the field $GF(2^3)$ because the polynomial exceeds degree $m$
- In a **prime field** we must do modular reduction by $p$ as that is what we are working with, we must reduce the intermediate result by an **irreducible polynomial instead**
- We know to stop polynomial long division when the elements have degree $< m$ 
- $P(x) = x^3 + x + 1$ 
- $A(x) \cdot B(x) = x^4 + x^3 + x + 1 \ (mod \ x^3 + x + 1)$ 
	![](Pasted%20image%2020260224222343.png)
	DO SOME PRACTICE

##### Inversion for extension fields
- We perform inversion like so:
- $A(x) \cdot A^{-1} \equiv 1 (mod \ P(x))$ 
- As with prime fields we can calculate the inverse via EAA


### AES' finite field
- AES uses the extension field GF(2^8) so up to 7 degree, for its polynomials 
- Operations are the same as those in other $GF(2^m)$ using the irreducible polynomial $P(x) = x^8 + x^4 + x^3 + x + 1$ 
	Multiplication and inversion is governed by this, naturally, and arithmetic is governed by XOR and the $a$ coefficients in the prime-sub field of the extension field
- Polynomials then in AES represent single bytes. 