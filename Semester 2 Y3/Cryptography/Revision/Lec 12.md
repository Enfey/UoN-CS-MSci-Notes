## Recap
- The group $\mathbb{Z}_p^*$ forms a cyclic group where $\circ$ is $\times$ 
- We can use this to create cryptographic systems like Diffie-Hellman based on the discrete logarithm problem cyclic groups instantiate under certain conditions:
	$g^a = g \circ g \circ \dots \circ g \ mod \ p$ 
	Need to reverse the exponentiation to discover the shared secret, computing forward is cheap, but there is no quick way to reverse this for large primes. 
- Want another mathematical structure where we can define another "easy forward, hard backward " operation
- We look at EC, solving the ECDLP is believed to be much harder than the integer DLP, which also means equivalent security with much smaller key sizes. 


### Elliptic curve intuition
- A point $(x, y)$ is reached by moving $x$ horizontally, $y$ vertically.
- We restrict $x$ and $y$ to be only those points who satisfy the equation $x^2 + y^2 = r^2$ 
- Thus, we only keep those points whose distance from the origin is equal to $r$, which forms a circle from the plane.

### Ellipctic Curves
- By introducing co-efficients $ax^2 + by^2 = r^2$ one can deform the shape on the plane
	- We have pairs of x and y for which this equation holds defining points along the curve. 
	- $a$ and $b$ are fixed.
- If $a \gt b$ then movement in $x$ is more 'expensive' as it moves more; allowable $x$'s are more restricted than $y$ values, leading to a squashed shape horizontally as we are limited in $x$ 
- So instead of a circle, we get an **ellipse** where the points that construct this shape in the plane are the solution to the above equation
- One problem with this equation is that the elements on these curves are real numbers and are thus, infinite, we cannot work with arbitrary real co-ordinates in a cryptographically predictable way. 
	- Could have fractions of x and fractions of y and may have a radius of root 2, could be fuckin anything
- **NEED A CURVE WHERE THE SET OF POINTS FORMS A GROUP WITH A HARD DLP**

### Elliptic Curve Definition - Weierstrass Form
- An elliptic curve over $\mathbb{Z}_p$ is the complete set of points $(x, y)$ where both co-ordinates are in the group and satisfy the Weierstrass equation: $$y^2 \equiv x^3 + ax + b \ (mod \ p)$$
- We take $p>3$ 
- There is a neutral element $\mathcal{O}$ called the **point at infinity**
- We have the requirement that $4a^3 + 27^b \neq 0 \ (mod \ p)$ 
	- arises from the fact that an equation for the elliptic curve is a cubic polynomial in $x$ 
	- ...


### Elliptic curve example plotted over the reals
![](Pasted%20image%2020260322190620.png)
- $y^2 = x^3 + ax + b \ (mod \ p)$ 
- Notice the symmetry about the x-axis
	- The equation denoted above; we fix an x value and we get $y^2$ = some number
	- Squaring destroys the sign however. That is, if a given $y$ solves the equation, then its negation also solves the equation, as their squares are equal
	- So whenever $(x, y)$ is a solution, $(x, -y)$ is also automatically a solution using the exact same $x$ value, sitting at the same horizontal position but on the inverse side of the $x$ axis.


### Elliptic Curve Requirements
- For a **DLP problem** we need a cyclic group; trivially, these are just points (x,y) that solve the equation $y^2 \equiv x^3 + ax + b \ (mod \ p)$ 
- We have a group operation; $\circ$ and we take this operation to be **point addition**
	![](Pasted%20image%2020260322193812.png)
	$P+Q = R$ 
	For 2 arbitrary group elements. 

### Point addition
- Point addition is defined geometrically as drawing a straight line through $P$ and $Q$ and this will always intersect a third point on the curve, which is then reflected about the x-axis
	![](Pasted%20image%2020260322194050.png)
- Reflection gives us associativity
	- Without reflection, $(P + Q) + R$ and $P+(Q+R)$ would hit different intersection points
	- There is some proof about this
	- Reflection is permitted via the prior description about x-axis symmetry when solving for group membership, ie.., the equation. 

### Point doubling
- Point addition unto a single group element
- We cannot draw a unique line through a single point
- The solution is to take the tangent line which just touches the curve at $P$ and solve $P+P = 2P$ 
- The tangent line will intersect another point on the curve, which we also reflect about the x-axis. 
- ![](Pasted%20image%2020260322194613.png)


### Group laws
- **Closure** - adding any two points on the curve produces another point on the curve.
- **Associativity** - of the group/point addition operation, given via reflection
- **Neutral element** - point at infinity
- **Inverses** - every point $(x, y)$ has an inverse $(x, -y)$ which yields the point at infinity?
- **Commutativity** - $P+Q = Q+P$ this follows directly from the geometry, the line through them is identical and thus the third intersection point and subsequent reflection will therefore be identical.

THE POINT ADDITION OPERATION BUILDS UP NATURALLY, WE CAN ADD P TO DIFFERENT GROUP ELEMENTS
![](Pasted%20image%2020260522174439.png)


### Point addition equations
- We can derive equations for point addition based on the equation for a line that intersects the curve in three sep places 
- Given:
	- $y^2 = x^3 + ax + b$ 
	- Points $P = (x_1, y_1)$ and $Q = (x_2, y_2)$ 
	- Line $y = s \cdot x + m$ 
	- We find all the points where the equation for the line, and and the EC group requirement hold true
	- All the intersecting points in a group. 
- We essentially plug the bottom formula into the top one
- After some rearrangement we are left with the following equations:
	- $P+Q = (x_3, y_3)$ 
	- $x_3 = s^2- x_1 - x_2$ 
	- $y_3 = s(x_1 - x_3) - y_1$ 
	- ![](Pasted%20image%2020260322201335.png)
	- Included in the appendix of an exam


### Inverses
About the y axis, y^2 destroys the sign, so inverse is automatically a solution, works under any mod p. 
- The $y's$ are additive inverses of each other mod $p$, to obtain $-P$
		- $P=(x, y)$
		- $-P=(x, -y)$ 
When doing -P ensure do (x, p-y) where p is the prime modulus


### Neutral element
- We require that P plus its inverse yields the neutral element
- Doesn't have co-ordinates in practice
- In practice, detect when x-coordinate is the same, and the y values are inverses mod p, as this result will naturally yield the neutral element. 
	- Div by zero = point at infinity



### Cyclic groups
- The points on an elliptic curve form cyclic subgroups, via point addition unto themselves
- Bounce around valid co-ordinates of elliptic curve
- Eventually arrive at $aP$ after number of applications, which is exactly the discrete logarithm problem. 
	- $P$ is the generator that generates points on the elliptic curve; this has maximal order equivalent to the cardinality of the EC group. 

#### ECDLP
- Given a curve $E$, a generator $P$, and a point on the curve/in the group $aP$, what is $a?$ 
	$$aP = P + P \dots + P$$
- Implement via doubling e.g., if a was 13 then 1101, = $13 = 1 \times 8 + 1 \times 4 + 0 \times 2 + 1 \times 1$ 
- So $13P = 8P+4P+0P+1P$  which is computable via doubling, 3 doubles, 1p, 2p, 4p, 8p, and 2 additions to combine them
- 5 operations
 