## A recap of $\mathbb{Z}^*_p$ 
- The group $\mathbb{Z}^*_p$ forms a cyclic group with the group operation $\circ$ as $\times$ 
- We can use this to create cryptographic systems like Diffie-Hellman based on the 'hardness' of the Discrete Logarithm Problem. $$g^a = g \circ g \circ \dots \circ g$$
	$a$ times. Need to reverse the exponentiation to find $a$ to uncover the shared secret, which is visible in the last lecture. Computing the exponentiation forward is cheap, but there is no quick way reverse to this for large primes. 
- We'd like to find another mathematical structure where we can define another similar "easy forward, hard backward" operation.
- We look toward elliptic curves - solving the ECDLP is believed to be much harder than the originally DLP, meaning equivalent security with much smaller key sizes, for example, a 256 bit ECC key offers roughly the same security as 3072 bit RSA key.

## Elliptic Curve Intuition
- A point $(x, y)$ is reached by moving $x$ horizontally and $y$ horizontally.
- We restrict $x$ and $y$ via $x^2 + y^2 = r^2$ (via distance equation+some simplification)
- Thus we keep only the points whose distance from the origin equals $r$, which yields a circle from the plane. 
	![](Pasted%20image%2020260322181456.png)
	
## Elliptic Curves
- By introducing coefficients $ax^2 + by^2 = r^2$, the points yielded by the equation form a deformed shape on the plane
	- Have pairs of $(x, y)$ for which this equation holds defining points along the curve.
- If $a > b$ then movement in $x$ is expensive for solving the equation, leading to a squashed shape horizontally, and vice versa.
- So instead of a circle, we get an **ellipse**, where the points that construct this shape in the plane are the solution to so equation that $= r^2$ 
- One problem with this equation is that elements on these curves are real numbers - we need a finite, discrete set of elements, we cannot work with arbitrary real co-ordinates in a cryptographically predictable way. 
	- We can have fractions of $x$ and fractions of $y$ and may have a radius of root two. There is no restriction
- We seek to find some curve where the set of points forms a group with a hard DLP.

### Elliptic Curve Definition - Weierstrass Form
- An elliptic curve over $\mathbb{Z}_p$ is the complete set of points $(x, y)$ where both co-ordinates are $\in \mathbb{Z}_p$ and satisfy the Weierstrass equation:$$y^2 \equiv x^3 + ax + b \ (mod \ p)$$
- We take $p > 3$.
- There is a neutral element $\mathcal{O}$ called the point at infinity.
- We have the requirement that $4a^3 + 27^b \neq 0 \ (mod \ p)$ 
	- This requirement arises from the fact that the equation for an elliptic curve is a cubic polynomial in $x$.
	- Considered over the real numbers.
	- Like any polynomial, it may have repeated roots - these are some values of $x$ where it touches zero or some point multiple times. 
	- If there is a double root, the curve has a cusp - a sharp pointed spike.
	- If there is a triple root, the curve has self-intersects.
	- This breaks later operations we perform with the group operation; the expression above is exactly the discriminant of the cubic polynomial that equals zero precisely when the cubic has a repeated root.
		- We say that the cubic has no repeated roots if this equation holds, meaning the curve is smooth everywhere, therefore the tangent is well defined and point addition never fails. 

### Elliptic Curve Example Plotted Over $\mathbb{R}$ 
![](Pasted%20image%2020260322190620.png)
- $y^2 \equiv x^3 - 3x + 3$ over $\mathbb{R}$ 
- Notice the symmetry about the $x$ axis - this is algebraically fundamental
	- The equation is $y^2 = x^3 + ax + b$
	- When we fix an $x$ value and solve, we get $y^2 = number$ 
	- Squaring destroys the sign, both $+y$ and $-y$ produce the same $y^2$ 
	- So whenever $(x, y)$ is a solution, $(x, -y)$ is also automatically a solution using the exact same $x$ value, sitting at the same horizontal position but on the inverse side of the $x$ axis.
	- This gives us inverses of group elements where the elements are pairs, and we define the inverses over $y$.
	- This will also work under any $mod \ p$ 

### Elliptic Curve Requirements
- For a DLP problem, we need a cyclic group:
	- We need elements within the group; trivially, these are just points $(x, y)$ that satisfy the equation: $ $$y^2 \equiv x^3 + ax + b \ (mod \ p)$$
	- A group operation $\circ$; we take this operation to be **point addition**
		![](Pasted%20image%2020260322193812.png)$$P+Q = R$$
		For 2 arbitrary group elements/points on the curve.


#### Point addition
- Point addition is defined geometrically as drawing a straight line through $P$ and $Q$; this will always intersect a third point on the curve, which is then reflected about the $x-axis$ 
	![](Pasted%20image%2020260322194050.png)
	- Reflection gives us associativity of the group operation; without it, the point addition as a group operation would not be valid. 
		- Without reflection $(P+Q)+R$ and $P+(Q+R)$ hit different third intersection points when working through the geometry
		- There is some bullshit proof that means this reflection makes this operation associative that we don't need to know about lol.
	- Reflection is permitted via the prior description of $x-axis$ symmetry.

#### Point Doubling
- This is where point addition is unto a singular group element rather than unique ones. 
- We cannot draw a unique line through a single point in the normal sense.
- The solution is to take the tangent line, which just touches the curve, at $P$ instead to solve $P+P = 2P$ 
- The tangent line will intersect another point on the curve
- Again, we reflect this point about the $x-axis$ 
	![](Pasted%20image%2020260322194613.png)

#### Group Laws
- Closure - adding any two points on the curve produces another point on the curve. The addition formula guarantees, further note  intersection+reflection. 
- Associativity of the group/point addition operation. This is given via reflection, as described above.
- Neutral element - covered next, this is the point at infinity.
- Inverses - every point $(x, y)$ has inverse $(x, -y)$. This was described previously
- Commutativity - $P+Q = Q+P$ this follows directly from the geometry, the line through $P$ and $Q$ is identical to the line through $Q$ and $P$, so the third intersection point is the same regardless.
- The point addition operation builds up naturally - we can add $P$ to different numbers
	- $2P + P = 3P$ 
	- $3P+P = 4P$
	- $2P + 2P = 4P$ 
	![](Pasted%20image%2020260322195523.png)
	![](Pasted%20image%2020260322195555.png)
	![](Pasted%20image%2020260322195628.png)
	

### Point Addition Equations
- We can derive equations for point addition based on the equation for a line that intersects the curve in three separate places
- Given:
	- $y^2 = x^3 + ax + b$ 
	- Points $P = (x_1, y_1)$ and $Q = (x_2, y_2)$ 
	- Line $y = s \cdot x + m$ 
	- We find all the points where both $y = s \cdot x +m$ and $y^2 = x^3 + ax + b$ 
	- All the intersecting points in a group
- We essentially plug the bottom formula into the top one:
	- $(sx+m)^2 = x^3 + ax+b$ 
	- $s^2x^2+2sxm + m^2 = x^3 + ax + b$ 
		- From the standard algebraic identity $(A+B)^2 = A^2 + 2AB + B^2$
- After some rearrangement, which we do not cover here, we are left with the following equations:
	- $P+Q = (x_3, y_3)$ 
	- $x_3 = s^2 - x_1 - x_2$ 
	- $y_3 = s(x_1 - x_3) - y_1$ 
	- ![](Pasted%20image%2020260322201335.png)
	- Change in y over change in x under mod prime in the group, when point addition, if point doubling however, the slope of the line changes to the tangent of the line, naturally.
	- This will be included in the appendix of the exam. 

#### Example - Point addition
- $y^2 \equiv x^3 + 2x + 2 \ mod \ 17$
- $(3, 1) + (9, 16)$ 
- Point addition, not doubling, therefore top equation:
	- $s = \frac{16-1}{9-3} = \frac{15}{6}$
	- $= 15 \cdot 6^{-1} = 15 \cdot 3 \ mod \ 17$ 
	- $=11$
- $x^3 = 11^2 - 3 - 9 = 109 \ mod \ 17 \equiv 7$ 
- $y^3 = 11(3-7)-1 = 11 \cdot 13 - 1 \equiv 6 \ mod \ 17$ 
	- The 13 comes from $-4$ mod $17$ - can leave in the -4 term and multiply out, but the same result will be achieved at the end


#### Example - Point Doubling
- Do another time whilst revising. 

### Inverses
- Described prior, the inverse $-P$ of a point $P$ is the point prior to its reflection about the $x$-axis. 
	- The equation is $y^2 = x^3 + ax + b$
	- When we fix an $x$ value and solve, we get $y^2 = number$ 
	- Squaring destroys the sign, both $+y$ and $-y$ produce the same $y^2$ 
	- So whenever $(x, y)$ is a solution, $(x, -y)$ is also automatically a solution using the exact same $x$ value, sitting at the same horizontal position but on the inverse side of the $x$ axis.
	- This gives us inverses of group elements where the elements are pairs, and we define the inverses over $y$.
	- This will also work under any $mod \ p$ 
		- - The $y's$ are additive inverses of each other mod $p$, to obtain $-P$
		- $P=(x, y)$
		- $-P=(x, -y)$ 
	![](Pasted%20image%2020260322204340.png)
- It is not possible on an elliptic curve to have 2 points with the same $x$ co-ordinate that are not inverses of one another with respect to $y$ 
### Neutral Element - The point $\mathcal{O}$ at Infinity
- We require that $P+(-P) =$ **neutral element**
	- This is of course, as visible below, a straight vertical line with infinite points(for elliptic curve plotted with reals) denoting the reflection about the $x$ axis.
		![](Pasted%20image%2020260322204626.png)
- The point at infinity is the neutral element on an elliptic curve
	- $P+(-P) = \mathcal{O}$ 
	- $P+\mathcal{O}$ 
		- If imagine taking the third intersection and reflecting it, lands back at $P$ again, somewhat works. 
		- Very handwavey!
- In practice this point doesn't have co-ordinates, and cannot be used within the normal formula?
- When implementing, must detect when the $x$ value are equal, and the $y$ values are inverses mod $p$ (when added), as this result will naturally yield the neutral element.
	- $y^2 \equiv x^3 + 2x + 2 \ (mod \ 17)$ 
	- (7, 6) + (7, 11)
	- $\frac{11-6}{7-7} = \frac{5}{0} = \mathcal{O}$ 


### Cyclic groups
- The points on an elliptic curve, including the neutral element $\mathcal{O}$ form cyclic subgroups.
	![](Pasted%20image%2020260322205629.png)
- Add $P$ to itself to get $2P$, then $2P+P$ etc
- We 'bounce' around the valid co-ordinates of the elliptic curve via the group operation point addition
- We eventually arrive at $aP$ after a number of applications of this. To break this, we must ask how many times we did $P \circ P \circ \dots \circ P$ which is exactly the discrete logarithm problem. 
	- Immune to index calculus.
	- $P$ is the generator that generates points on the elliptic curve; as with multiplicative prime groups, the generator should have maximal order equivalent to the cardinality of the EC group. 
		- It should generate the entire elliptic curve/single cyclic group, though this is only met under certain conditions. 

#### Elliptic Curve DLP
- Given a curve $E$, a primitive root/generator $P$, and a point on the curve/in the group $aP$, what is $a$?
- This is the Elliptic Curve Discrete Logarithm Problem $$aP = P + P \dots + P$$
- We do not literally add $P$ to itself $a$ times, we use double and add
	- We write $a$ in binary, e.g., if $a$ was $13$ then $1101_2$ 
	- When a $1$ appears, compute with respect to the binary
		- $13 = 1 \times 8 + 1 \times 4 + 0 \times 2 + 1 \times 1$ 
		- So $13P = 8P+4P+0P+1P$ 
		- Just need to compute these terms via doubling - 3 doublings - $P->2P->4P->8P$ , and then 2 additions to combine them $8P+4P+P$
		- $5$ operations, which makes it feasible to go forwards.

![](Pasted%20image%2020260322211040.png)
- In saying prime fields, we are being precise about the arithmetic foundation we are working in.
- The elliptic curve equation under the modulo of the prime field forms a cyclic group with the group operation being defined as point addition
- The prime field sits above this defining addition and multiplication over that same modulus such that equations like point addition can function. 
- The group is a construction within the prime field. 
