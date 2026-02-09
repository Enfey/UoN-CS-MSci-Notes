# Transformations
> A transformation is any operation that takes a vector and produces a new vector.

## Translation
> Translation is the operation which moves a point. 

For example, a point represented as a vector $v$:
$v = (3, 2)$ 
can be translated $1$ in the $x$ and $2$ in the $y$, which is represented as the translation vector $t = (1, 2)$ 

Translation is calculated by vector addition. For the above, $u = (3+1, 2+2) = (4, 4)$
![](Pasted%20image%2020260208234441.png)


### Application
Given a triangle made from three vertices $v_0, v_1, v_2$ 
$v_0 = (0,1)$ 
$v_1 = (1, 1)$ 
$v_2 = (1, 0)$ 
![](Pasted%20image%2020260208234552.png)
Each vertex can be translated, for example, if the triangle above needs to be moved downwards a distance y, then subtract y from each of vertices' y component. 

So we define $t = (0, -1.5)$ 

$v_0 = \begin{pmatrix} 0 \\ 1 \end{pmatrix} + \begin{pmatrix} 0 \\ -1.5 \end{pmatrix} = \begin{pmatrix} 0 \\ 1 - 1.5 \end{pmatrix} = \begin{pmatrix} 0 \\ -0.5 \end{pmatrix}$ 
$v_1 = \begin{pmatrix} 1 \\ 1 \end{pmatrix} + \begin{pmatrix} 0 \\ -1.5 \end{pmatrix} = \begin{pmatrix} 1 + 0 \\ 1 - 1.5 \end{pmatrix} = \begin{pmatrix} 1 \\ -0.5 \end{pmatrix}$
$v_2 = \begin{pmatrix} 1 \\ 0 \end{pmatrix} + \begin{pmatrix} 0 \\ -1.5 \end{pmatrix} = \begin{pmatrix} 1 + 0 \\ 0 - 1.5 \end{pmatrix} = \begin{pmatrix} 1 \\ -1.5 \end{pmatrix}$ 
![](Pasted%20image%2020260208234852.png)

## Rotation
*Rotation* is the operation which rotates a **direction vector** around a specified angle. 
The vector $v$ can be rotated by an angle $\theta$ radians as follows:
	$rot_{\theta}(v) = (cos \theta * v_0 - sin \theta * v_1, sin \theta * v_0 + cos \theta * v_1)$ 
The vector $v$ can be rotated by an angle $\theta = 0.79 \ radians$, 45 degrees as follows:
	$v = (0, 1)$ 
	$u = rot_{\theta}(v) = (0.71 * 0 - 0.71 * 1, 0.71 * 0 + 0.71 * 1) = (-0.71, 0.71)$ 

![](Pasted%20image%2020260209000152.png)

![](Pasted%20image%2020260209001048.png)
![](Pasted%20image%2020260209000305.png)
This becomes fairly obvious in the context of the unit circle, where the angle is declared via the quadrant and the terminal arm, and the point to go to is determined by cos and sin, given that the hypotenuse is 1, the equations simplify, multiplying to get to where the angle is pointing, for a direction vector.

Rotation works in an anti-clockwise manner. 
### Application
Given a triangle is made up of three vertices:
$v_0 = \begin{pmatrix} 0 \\ 1 \end{pmatrix} \quad v_1 = \begin{pmatrix} 1 \\ 1 \end{pmatrix} \quad v_2 = \begin{pmatrix} 1 \\ 0 \end{pmatrix}$ 
Each vertex can be rotated. For example, if the triangle above is rotated anti-clockwise 90 degrees, each vertex is rotated anti-clockwise 90 degrees. 

$v_0 = \text{rot}_\theta \begin{pmatrix} 0 \\ 1 \end{pmatrix} = \begin{pmatrix} \cos 1.57 \cdot 0 - \sin 1.57 \cdot 1 \\ \sin 1.57 \cdot 0 + \cos 1.57 \cdot 1 \end{pmatrix} = \begin{pmatrix} 0 \cdot 0 - 1 \cdot 1 \\ 1 \cdot 0 + 0 \cdot 1 \end{pmatrix} = \begin{pmatrix} -1 \\ 0 \end{pmatrix}$ 
$v_1 = \text{rot}_\theta \begin{pmatrix} 1 \\ 1 \end{pmatrix} = \begin{pmatrix} \cos 1.57 \cdot 1 - \sin 1.57 \cdot 1 \\ \sin 1.57 \cdot 1 + \cos 1.57 \cdot 1 \end{pmatrix} = \begin{pmatrix} 0 \cdot 1 - 1 \cdot 1 \\ 1 \cdot 1 + 0 \cdot 1 \end{pmatrix} = \begin{pmatrix} -1 \\ 1 \end{pmatrix}$ 
$v_2 = \text{rot}_\theta \begin{pmatrix} 1 \\ 0 \end{pmatrix} = \begin{pmatrix} \cos 1.57 \cdot 1 - \sin 1.57 \cdot 0 \\ \sin 1.57 \cdot 1 + \cos 1.57 \cdot 0 \end{pmatrix} = \begin{pmatrix} 0 \cdot 1 - 1 \cdot 0 \\ 1 \cdot 1 + 0 \cdot 0 \end{pmatrix} = \begin{pmatrix} 0 \\ 1 \end{pmatrix}$ 
![](Pasted%20image%2020260209001157.png)
Relatively obvious, given $(x, y)$ of 90 degrees on the unit circle. 

## Scale
Applying a scale transformation stretches of shrinks a vector by a scalar. The vector $v$ can be stretched by a scalar $s$ in each dimension independently as follows:

$v \cdot s = (v_1 \cdot s_1, v_2 \cdot s_2)$ 

For example, if we have the vector:
$v = (4, 2)$ 
And we want to scale half in the $x$ dimension, and $1.5$ in the y dimension:
$s = (0.5, 1.5)$ 
$v \cdot s = (4 \cdot 0,5, 2 \cdot 1.5) = (2, 3)$ 

![](Pasted%20image%2020260209002119.png)

### Application
Given a triangle is made up of three vertices.
$v_0 = \begin{pmatrix} 0 \\ 1 \end{pmatrix} \quad v_1 = \begin{pmatrix} 1 \\ 1 \end{pmatrix} \quad v_2 = \begin{pmatrix} 1 \\ 0 \end{pmatrix}$ 
Each vertex can be scaled. For example, if the triangle above needs to be scaled to make it half as wide, each vertex is scaled by $s = (0.5, 1.0)$ 

$v_0 \cdot s = \begin{pmatrix} 0 \\ 1 \end{pmatrix} \begin{pmatrix} 0.5 \\ 1 \end{pmatrix} = \begin{pmatrix} 0 \cdot 0.5 \\ 1 \cdot 1 \end{pmatrix} = \begin{pmatrix} 0 \\ 1 \end{pmatrix}$ 
$v_1 \cdot s = \begin{pmatrix} 1 \\ 1 \end{pmatrix} \begin{pmatrix} 0.5 \\ 1 \end{pmatrix} = \begin{pmatrix} 1 \cdot 0.5 \\ 1 \cdot 1 \end{pmatrix} = \begin{pmatrix} 0.5 \\ 1 \end{pmatrix}$ 
$v_2 \cdot s = \begin{pmatrix} 1 \\ 0 \end{pmatrix} \begin{pmatrix} 0.5 \\ 1 \end{pmatrix} = \begin{pmatrix} 1 \cdot 0.5 \\ 0 \cdot 1 \end{pmatrix} = \begin{pmatrix} 0.5 \\ 0 \end{pmatrix}$ 

![](Pasted%20image%2020260209002425.png)

This does not move the points per say but merely stretches/shrinks the original vectors. 


## Transformation matrices
> A transformation matrix is a 4x4 matrix which represents a transformation

Rotations and scaling are linear transformations, translation is not, it does not map the zero vector to itself. As a result, it cannot be represented via a 3x3 matrix. This means we embed 3D points into 4D space by attaching an element, w, =1 to each co-ordinate in 3D space. 
	$(x,y,z) \to (x, y, z, 1)$ 

### Translation matrix
> The translation matrix $T$ which translates by the vector $t = (t_x, t_y, t_z)$ is as follows.

$T(t) = \begin{pmatrix} 1 & 0 & 0 & t_x \\ 0 & 1 & 0 & t_y \\ 0 & 0 & 1 & t_z \\ 0 & 0 & 0 & 1 \end{pmatrix}$ 
### Rotation matrix
> The three rotation matrices around each axis by the angle $\theta$ in radians, are $R_x, R_y,$ and $R_x$ as follows. 

$R_x(\theta) = \begin{pmatrix} 1 & 0 & 0 & 0 \\ 0 & \cos(\theta) & -\sin(\theta) & 0 \\ 0 & \sin(\theta) & \cos(\theta) & 0 \\ 0 & 0 & 0 & 1 \end{pmatrix}$ 

$R_x(\theta) = \begin{pmatrix} 1 & 0 & 0 & 0 \\ 0 & \cos(\theta) & -\sin(\theta) & 0 \\ 0 & \sin(\theta) & \cos(\theta) & 0 \\ 0 & 0 & 0 & 1 \end{pmatrix}$

$R_z(\theta) = \begin{pmatrix} \cos(\theta) & -\sin(\theta) & 0 & 0 \\ \sin(\theta) & \cos(\theta) & 0 & 0 \\ 0 & 0 & 1 & 0 \\ 0 & 0 & 0 & 1 \end{pmatrix}$ 
### Scale matrix
> The scale matrix $S$ which scales by the vector $s = (s_x, s_y, s_z)$ is as follows.

$S(s) = \begin{pmatrix} s_x & 0 & 0 & 0 \\ 0 & s_y & 0 & 0 \\ 0 & 0 & s_z & 0 \\ 0 & 0 & 0 & 1 \end{pmatrix}$

The derivation(s) make sense if you compose a general matrix and deform that into the general operation we want to apply. 
### Applying a transformation
$\begin{pmatrix} m_{0,0} & m_{0,1} & m_{0,2} & m_{0,3} \\ m_{1,0} & m_{1,1} & m_{1,2} & m_{1,3} \\ m_{2,0} & m_{2,1} & m_{2,2} & m_{2,3} \\ m_{3,0} & m_{3,1} & m_{3,2} & m_{3,3} \end{pmatrix} \times \begin{pmatrix} p_0 \\ p_1 \\ p_2 \\ 1 \end{pmatrix} = \begin{pmatrix} p_0 m_{0,0} + p_1 m_{0,1} + p_2 m_{0,2} + m_{0,3} \\ p_0 m_{1,0} + p_1 m_{1,1} + p_2 m_{1,2} + m_{1,3} \\ p_0 m_{2,0} + p_1 m_{2,1} + p_2 m_{2,2} + m_{2,3} \\ p_0 m_{3,0} + p_1 m_{3,1} + p_2 m_{3,2} + m_{3,3} \end{pmatrix}$
Takes the form of matrix-vector multiplication. 3D space is represented by 3-element vectors. A **homogeneous co-ordinate** is an additional 4th element of a 3D vector. It is also called the $w$ component, and a homogenous vector is $p = (p_x, p_y, p_z, p_w)$. Application of transformation matrices is applied to homogenous vectors. 


## Geometric interpretations of linear combinations
The standard basis of $\mathbb{R}^3$ is 3 vectors: $v_1, v_2, v_3 \in \mathbb{R}^3$ 
$v_1 = (1, 0, 0)$ 
$v_2 = (0, 1, 0)$ 
$v_3 = (0, 0, 1)$ 
Any point in $\mathbb{R}^3$ can be represented as a *linear combination* of these 3 standard basis vectors with 3 scalar values: $s_1, s_2, s_3$ 

For example, the point $p = (1, 2, 1)$ is represented as $s_1 = 1, s_2 = 2, s_3 = 1$, and a linear combination of these scalars with the basis vectors of $\mathbb{R}^3$ results in $p$.
$p = 1(1, 0, 0) + 2(0, 1, 0) + 1(0, 0, 1)$
$p = (1, 0, 0) + (0, 2, 0) + (0, 0, 1)$ 
$p = (1, 2, 1)$ 

We can represent this geometrically as follows.  
	We start with the vector $s_1v_1 = 1(1, 0, 0) = (1, 0, 0)$ 
		![](Pasted%20image%2020260209013332.png)
	Then we add the vector $s_2v_2 = 2(0, 1, 0) = (0, 2, 0)$ 
		![](Pasted%20image%2020260209013439.png)
	Then add the vector $s_2v_2 = 1(0,0,1) = (0, 0, 1)$ 
		![](Pasted%20image%2020260209013520.png)
		
### Identity transformation matrix as basis vectors + translation
 $I = \begin{pmatrix} 1 & 0 & 0 & 0 \\ 0 & 1 & 0 & 0 \\ 0 & 0 & 1 & 0 \\ 0 & 0 & 0 & 1 \end{pmatrix}$ 
This is known as the identity transformation matrix.

With column vectors:
	The first column $X$ usually called $e_1$ is a vector which points in the $x$ direction $(1, 0, 0)$.
	The second column $Y$, usually called $e_2$ is a vector which points in the $y$ direction $(0, 1, 0)$ 
	The third column $Z$ usually called $e_3$ is a vector which points in the $z$ direction $(0, 0, 1)$ 
	The fourth column $T$ is a translation vector $(0, 0, 0)$
The $w$ element of these four vectors, seen as the $4^{th}$ row of the matrix above, is a homogenous coordinate, which differentiates between a vector representing a position and a vector representing a direction. The homogeneous coordinate of a direction is 0, as can be seen in the case of $e_1, e_2,$ and $e_3$ since the basis vectors are directions, and the homogeneous co-ordinate of a position is $1$, in this case for $T$. 

If we transform a position vector $p$, after the transformation, the new position vector is a linear combination of the co-ordinates of $p$ with the basis vectors stored in the matrix. 
	![](Pasted%20image%2020260209014907.png)
	Thus, essentially reconstructs the same point from the standard basis, plus zero translation. 
	The same linear combination idea applies regardless of matrix form. 

## Combining transformations
It is possible to combine transformations and store the result in a new transformation matrix. This is achieved via matrix-matrix multiplication of two transformation matrices. 

## Understanding transformation combinations

## Order of the transformations