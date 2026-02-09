# Vectors and Matrices
- In 3D computer graphics we primarily use vectors in $\mathbb{R}^2$ and in $\mathbb{R}^3$. If we have a vector $v$, each of the real number components $v_1, v_2, \dots, v_n$ are called *co-ordinates.*
- A **vector** can be written in *row-vector* notation
	$v = (v_1, v_2, \dots, v_n)$ 
- Or in *column-vector* notation
	$$\begin{pmatrix} v_{1} \\ v_{2} \\ \dots \\v_{n} \\\end{pmatrix}$$
- Vectors can be represented geometrically as arrows, e.g., in $\mathbb{R}^2$ on the Euclidean XY-plane, the vector $v = (2, 1)$ can be shown as 
	![](Pasted%20image%2020260208193859.png)
- We can represent vectors in $\mathbb{R}^3$ geometrically in 3-space.
- A vector can be used to represent the *position* of a point, where the base of the vector is placed at the origin of the concerned space, and the head of the vector is the position of the point. 
	- This is how vertex positions are interpreted.
- A vector can also be used to represent a *direction*, in which case the base of the vector can be placed anywhere, and the vector will point in a direction.
	- ![](Pasted%20image%2020260208194516.png)
	- Describe a relative displacement with the co-ordinates, rather than a fixed location. This would still yield the same direction regardless of the base. 
	- Any two vectors that are scalar multiples of one another represent the same direction. 
## Vector Mathematics
### Vector Scale
> Multiplying all elements in the vector independently by a scalar $s \in \mathbb{R}^n$ 

$sv = s (v_1, v_2, \dots, v_n) = (s \cdot v_1, s \cdot v_2, \dots, s \cdot v_n)$ 
![](Pasted%20image%2020260208194832.png)
### Vector Addition
> Performed by adding corresponding elements, element wise from each vector.

$u+v = (u_1, u_2, \dots, u_n) + (v_1, v_2, \dots, v_n) = (u_1 + v_1, u_2 + v_2, \dots u_n + v_n)$ 
![](Pasted%20image%2020260208195229.png)
As an example. We show geometrically, that vector addition essentially means apply $v$ after $u$, which is why they are added head to tail. 


### Linear Combination
> If we have $m$ vectors in $\mathbb{R}^n$ $v_1, v_2, \dots, v_n$ and $m$ scalars $s_1, s_2, \dots, s_n$ then a *linear combination* will scale each vector $s_iv_i$ and add all of the results to make a resulting vector $u \in \mathbb{R}^n$ as follows:

$u = s_1v_1 + s_2v_2 + \dots + s_mv_m$ 

![](Pasted%20image%2020260208195555.png)

### Vector Subtraction
> Vector subtraction of two vectors $v$ and $u$ is performed by subtracting elements, element wise, from each vector. 

$u-v = (u_1, u_2, \dots, u_n) - (v_1, v_2, \dots, v_n) = (u_1 - v_1, u_2 - v_2, \dots, u_n - v_n)$
![](Pasted%20image%2020260208200136.png)

### Vector length
> Vector length can be calculated using Pythagoras' method, for a vector $v$ 

$||v|| = \sqrt{v_1^2+v_2^2+\dots+v_n^2}$ 
![](Pasted%20image%2020260208200750.png)

### Vector normalisation
> *Vector* normalisation is used for making a *unit* vector, which has length 1. The unit vector is pronounced *hat*, e.g., $\hat{u}$ is u-hat.

$\hat{u} = u \cdot \frac{1}{||u||}$ 
![](Pasted%20image%2020260208201040.png)
Ignore 3/3.61 etc, just calc 1/3.61 and treat as regular scalar.
### Vector dot product
> *Vector* dot product, also called inner product, of two vectors $u$ and $v$ is defined as follows.

$u \cdot v = \sum^n_{i=1} u_{i} * v_{i}$
![](Pasted%20image%2020260208202508.png)

A positive dot product means the vectors make an acute angle. 
![](Pasted%20image%2020260208202531.png)
A negative dot product means the vectors make an obtuse angle
![](Pasted%20image%2020260208202551.png)
If one vector $\hat{u}$ is a unit vector, the dot product projects a second vector onto the direction of $\hat{u}$.
![](Pasted%20image%2020260208203021.png)
![](Pasted%20image%2020260208203030.png)


### Vector cross product
> The *vector cross-product* of two vectors in $\mathbb{R}^3$ is defined as:

$\mathbf{u} \times \mathbf{v} = \begin{pmatrix} (u_2 \cdot v_3) - (u_3 \cdot v_2) \\ (u_3 \cdot v_1) - (u_1 \cdot v_3) \\ (u_1 \cdot v_2) - (u_2 \cdot v_1) \end{pmatrix}$ 
Returning another vector $q \in \mathbb{R}^3$ that is perpendicular to the plane on which the original vectors lie (which are not scalar multiples, linearly independent, and point in different directions).
![](Pasted%20image%2020260208203812.png)


## Matrices
A *matrix* $M$ is a grid of scalars, with $m$ rows and $n$ columns.
$M = \begin{pmatrix} m_{1,1} & m_{1,2} & \cdots & m_{1,n} \\ m_{2,1} & m_{2,2} & \cdots & m_{2,n} \\ \vdots & \vdots & \ddots & \vdots \\ m_{m,1} & m_{m,2} & \cdots & m_{m,n} \end{pmatrix}$ 
A matrix with 1 row, is a row-vector $(m_{1,1}, m_{1,2}, \ldots, m_{1,n})$
A matrix with 1 column, is a column-vector$\begin{pmatrix} m_{1,1} \\ m_{2,1} \\ \vdots \\ m_{m,1} \end{pmatrix}$ 

### Matrix maths
#### Matrix transposition
$M^T$ means the transpose of matrix $M$. During this operation, the rows become the columns, and the columns become the rows.

$$M^T = \begin{pmatrix} m_{1,1} & m_{2,1} & \cdots & m_{m,1} \\ m_{1,2} & m_{2,2} & \cdots & m_{m,2} \\ \vdots & \vdots & \ddots & \vdots \\ m_{1,n} & m_{2,n} & \cdots & m_{m,n} \end{pmatrix}$$

#### Matrix-Vector Multiplication
If we have an $m \times n$ matrix $M$ and a vector $v \in \mathbb{R}^n$ then we can perform *Matrix-Vector Multiplication*, and the resulting vector $u$ is in $\mathbb{R}^m$. 
$$M = \begin{pmatrix} m_{1,1} & m_{1,2} & \cdots & m_{1,n} \\ m_{2,1} & m_{2,2} & \cdots & m_{2,n} \\ \vdots & \vdots & \ddots & \vdots \\ m_{m,1} & m_{m,2} & \cdots & m_{m,n} \end{pmatrix} \quad v = \begin{pmatrix} v_1 \\ v_2 \\ \vdots \\ v_n \end{pmatrix}$$

Becomes:
$$u = Mv = \begin{pmatrix} m_{1,1} & m_{1,2} & \cdots & m_{1,n} \\ m_{2,1} & m_{2,2} & \cdots & m_{2,n} \\ \vdots & \vdots & \ddots & \vdots \\ m_{m,1} & m_{m,2} & \cdots & m_{m,n} \end{pmatrix} \begin{pmatrix} v_1 \\ v_2 \\ \vdots \\ v_n \end{pmatrix} = \begin{pmatrix} v_1 m_{1,1} + v_2 m_{1,2} + \cdots + v_n m_{1,n} \\ v_1 m_{2,1} + v_2 m_{2,2} + \cdots + v_n m_{2,n} \\ \vdots \\ v_1 m_{m,1} + v_2 m_{m,2} + \cdots + v_n m_{m,n} \end{pmatrix}$$ As can be seen, the vector $v$ and each row $i$ of $M$ have a dot product operation applied, and the result is placed as co-ordinate $i$ in the new vector $u$. 

If we denote the columns of $M$ as vectors:


$$M = \begin{pmatrix} | & | & & | \\ m_1 & m_2 & \cdots & m_n \\ | & | & & | \end{pmatrix}$$
Then the result of $Mv$ is a linear combination:
$$u = Mv = \begin{pmatrix} | & | & & | \\ m_1 & m_2 & \cdots & m_n \\ | & | & & | \end{pmatrix} \begin{pmatrix} v_1 \\ v_2 \\ \vdots \\ v_n \end{pmatrix} = v_1 m_1 + v_2 m_2 + \cdots + v_n m_n$$
#### Matrix-Matrix Multiplication
If $M$ is an $m \times n$ matrix, and $N$ is an $n \times p$ matrix, then $M$ and $N$ can be multiplied, and the result is an $m \times p$ matrix. 
##### Row-column matrix multiplication
Each column $j$ of $N$ and each row $i$ of $M$ have a dot product operation applied, and the result is placed at $i, j$ element in the resulting matrix. 

![](Pasted%20image%2020260208214949.png)

##### Column-by-Column matrix multiplication
If we denote the columns of $N$ by vectors $v_1, v_2, \ldots, v_p$, then the result of multiplying $M$ and $N$ is the matrix-vector multiplication of $M$ with each column of $N$. 

$$MN = \begin{pmatrix} | & | & & | \\ Mv_1 & Mv_2 & \cdots & Mv_p \\ | & | & & | \end{pmatrix}$$
This is known as the column-by-column matrix-multiplication definition.

![](Pasted%20image%2020260208215400.png)

In general, matrix-matrix multiplication is not commutative, $MN \neq NM$.
# Trigonomietry
The unit circle is a circle with a radius of 1, centered at the origin $(0,0)$ on a cartesian plane. For any point $(x, y)$ on the circle, the co-ordinates are $p_x = cos \ \alpha$ and $p_y = sin \ \alpha$ respectively, given the hypotenuse is always $1$ for the unit circle. 

So, we pick a direction, and rotate by an angle $\alpha$, and land somewhere on the ricle of radius 1, taking $cos \ \alpha$ and $sin \ \alpha$ yields a unit vector $\hat{u}$ describing the position, as the distance from the origin $\sqrt{(cos \ \alpha)^2 + (sin \ \alpha)^2} = 1$ 

![](Pasted%20image%2020260208232637.png)


![](Pasted%20image%2020260208232649.png)

