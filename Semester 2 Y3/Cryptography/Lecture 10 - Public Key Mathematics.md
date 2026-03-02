## Modular Inversion
- In general arithmetic, the inverse of $a$ means we want a number $a^{-1}$ such that: $$a \cdot a^{-1} = 1$$
- In modular arithmetic, and more specifically, in Rings and prime fields(as a result of being constituted of a multiplicative group), a multiplicative inverse may exist such that: $$a \cdot a^{-1} \equiv 1 \ (mod \ p)$$
- Modular inverses exist when $gcd(a, p) = 1$ (particularly important for prime fields).
	- For prime fields, $gcd(a, p) = 1$, $\forall a \neq 0 \in GF(p)$.
- The **Euclidean Algorithm** can calculate $gcd(a,b)$, the **Extended Euclidean Algorithm** can calculate $a^{-1}$.
## Euclidean Algorithm
- The Euclidean algorithm calculates the greatest common divisor of two numbers $gcd(r_0, r_1)$ 
- $gcd(r_0, r_1) = 1$ denotes that $r_0$ and $r_1$ are co-prime. 
- ![](Pasted%20image%2020260301231708.png)
	- Intuition, that can compute the difference repeatedly until get to 0, and the last nonzero remainder is the $gcd$
	- We replace our difference with a modular reduction (which is must faster), and give it a new term. 
	![](Pasted%20image%2020260301214428.png)
	![](Pasted%20image%2020260301214435.png)
- $r_0$ is made of $q_1$ chunks of size $r_1$ and a leftover $r_2$.
- Any number $d$ that divides both $r_0$ and $r_1$ must also divide the leftover. 


### Bezout's Identity
- Bezout's identity tells us that the $gcd$ of two numbers can be expressed as the sum of multiples of these numbers: $$gcd(r_0, r_1) = s \cdot r_{0} + t \cdot r_{1}$$
### Extended Euclidean Algorithm
just rewatch the lecture and do the worksheet, fairly straightforward.
![](Pasted%20image%2020260301233157.png)
read top 3 first, then second lot of top 3, and then second lot, then see end of each respective algorithm.
![](Pasted%20image%2020260301233249.png)
first is always 1 in each r0 amount in EEA. 
r4 has first term 1.ro - 3.r1 because it is r2 !

![](Pasted%20image%2020260301234328.png)
