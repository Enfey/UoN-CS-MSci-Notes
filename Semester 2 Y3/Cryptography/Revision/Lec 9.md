## Modular inversion
- Recall that in general arithmetic the inverse of $a$ means that (for multiplication) we seek a number $a^{-1}$ such that $a \cdot a^{-1}  = 1$ 
- For modular arithmethic, which appears in groups, rings, and fields, there may be a modular multiplicative inverse such that:$$a \cdot a^{-1} \equiv 1 \ (mod \ p)$$
- We have a modular inverse when $gcd(a, p) = 1$, particularly important for prime fields as $gcd(a, p) = 1$, $\forall a \neq 0 \in GF(p)$ 
- The **Euclidean Algorithm** can calculate gcd, EEA calculates the multiplicative inverse of a member of a given algebraic structure.

## Euclidean algorithm
- The Euclidean algorithm calculates GCD of two numbers $gcd(r0, r1)$ 
- If this result is 1, the numbers are said to be **co-prime**
- ![](Pasted%20image%2020260301231708.png)
IDK HOW IT WORKS, LETS JUST DO THE FUCKIN ALGORITHM

Convert $r_0$ into the form $r_0 = q \cdot r_1 + r_2$ 

E.g., 