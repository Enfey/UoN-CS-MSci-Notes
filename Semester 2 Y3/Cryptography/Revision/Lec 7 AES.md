## Advanced Encryption Standard
- Specification for the encryption of data following DES
- Variant of the **rijndael** block cipher
	- SP-network with 128-bit block size, key length of 128, 192, 256-bits
	- Recall these are block ciphers whose rounds consist of substitution via s-boxes and permutation mixed with subkeys to achieve confusion/diffusion, respectively. 
- 10,12,14 rounds
- Unlike DES, AES transforms the whole block each round
	![](Pasted%20image%2020260224225732.png)
	Note there is more rounds, and the key schedule changes depending on the key size. 


### AES rounds structure
- **Key expansion**
	- Each round receives own key generated from master key
	- 16 byte key for AES-128, word = 4 bytes, thus, 4 words per round key. 
		- Need to generate then, 44 words from the master key for 12 round keys
- Uses rounds of 4 layers, and a final round of 3 layers
	![](Pasted%20image%2020260225005142.png)
	As can see, have a subkey that is initially mixed into the plaintext, then have 3 operations, then XOR the next round key in again. 
	This continues for all rounds, except the last one, where this is no mixcolumns. 

### Representation and manipulation
- Bytes are represents as a 4x4 block called the **state**
	![](Pasted%20image%2020260225005424.png)
- For every byte in state, visit SubBytes lookup table, which takes as input 8 bits, and returns 8 bits as output, returning a different byte
- For each row in the state, the ShiftRow transformation performs a left rotation on each row whose index > 1 e.g., row 2 has 1 byte left rotation, row 3 has a 2 byte rotation etc. 
	![](Pasted%20image%2020260225010130.png)
- For each column in the state, the MixCoumns applies a fixed transformation to achieve diffusion, taking 4 byte chunks and mixing them together linearly.

### S-box
- The AES s-box is based around the multiplicative inverse of $8$ bit values in $GF(2^8)$ 
	![](Pasted%20image%2020260225012705.png)
	Split byte into two, interpret as polynomial, inverse is initial S-box entry. 
- We interpret bytes as a polynomial in $GF(2^8)$ and compute the multiplicative inverse via EAA for every byte from 0 to 255. 
	$A_i \cdot A_i^{-1} \equiv 1 \ mod (P(x))$ yielding the neutral element of the prime-extension field under this irreducible polynomial.
	- Decent in creating a strongly non-linear mapping; field inversion is non-linear in that it involves polynomial division, but 0 stays as a fixed point as it has no multiplicative inverse, which we want to avoid
- We say that:$$
B'_i =
\begin{cases}
0 & \text{if } i = 0 \\
A_i^{-1} & \text{if } i > 0
\end{cases}
$$
- The inverses $B'_i$ undergo an **affine transformation** to produce the final $S-box$ destroying any remaining mathematical structure
	- An affine transformation is a linear transformation plus a linear vector. 
		![](Pasted%20image%2020260225012858.png)
	Matrix is fixed, and incremented the values until they got the desirable $S-box$ such that $0$ no longer maps to zero which would not be strong against differential/linear cryptanalysis as it is a fixed point under the prime binary extension field as it has no modular multiplicative inverse.


### S-box porperties
- The $s-box$ is bijective, and is therefore an invertible 1:1 mapping for a given byte $b$ 
- It maintains no fixed points via the affine transformation, there is no $A_i$ for which $S(A_i) = A_i$ 
- There is no inverse fixed points, that is $S(A_i) \oplus A_i = FF$ should not be its own bitwise complement(remember to think of it as polynomial and plus the coefficients). The 1s in between reveal the difference
	- Simple algebraic structure. If even a few inputs behave like this, there would be linearity to exploit within the S-box, whose design is meant to be entirely non-linear and defeat linear cryptanalysis
- **It is difficult to perform linear cryptanalysis**
	- Over $GF(2)$ a linear combination just means $XOR$ which is a linear function over bits. 
	- For a given equation $x_1 \oplus x_2  = y_1$ we say that the XOR of two input bits equals one output bit
		![](Pasted%20image%2020260522012429.png)
	- We check for all possible 256 inputs to the $S-box$ how often this equation is true, if the $S-box$ behaves randomly, it should be 50% of the time. 
	- No way to represent the input bits and output bits as linear combinations that reveal bias in the s-box; cannot predict input given its output. 
	- Minimises the largest bias as if you do this for one s-bix the ability to perform linear cryptanalysis on the entire cipher will be extremely difficult/impossible.
- **DIFFERENTIAL PREVENTION**
	- We say there is no likely predictable output difference from some input difference $\Delta$ and is thus resistant to differential cryptanalysis; all possibilities are equally likely for each given byte; 4 bit portion, and it is nonp-linear. 


### AES Diffusion
- Diffusion in AES consists of two layers:
	1. **Shift rows**
	2. **Mix columns**
- Shift rows simply performs left rotation on each row whose index exceeds 1 e.g., left rotate 1 for row 2 etc.
	![](Pasted%20image%2020260225010130.png)
- Weak on its own because it localises the diffusion to the row, and does not affect bytes
- However, it is needed by mixcolumns to create global diffusion with respect to the s-box output; ensures columns do not stay independent. 

### MixColumns
- Performs a **linear mixing** of bytes within each column
- It does this by representing those bytes as polynomials in Galois $GF(2^8)$ then performing matrix-vector multiplication to yield a column vector denoting all the output columns
- All of the input bytes in a column influence all of the output bytes, as they are all multiplied in each step to yield individual vector components. 
	![](Pasted%20image%2020260225033847.png)
	 $C_0 = 02 \cdot A2 + 03 \cdot 0D + 01 \cdot 4C + 01 \cdot 25$
		These bytes are polynomials, and thus addition on them essentially resolves to XOR in accordance with the prime subfield $GF(2)$
	$02 \cdot A2=$
		- $x \cdot (x^7 + x^5 + x)$ 
			- 02 = 00000010
			- A2 = 10100010
		- $= x^8 + x^6 + x^2$ 
			- Must now do polynomial long division according to $x$as exceeds $GF(2^8)$ 
	 $=x^6 + x^2 + x^4 + x^3 + x + 1$ 
			- This was acquired via a shortcut, where x^8 was substituted in accordance with the rearrangement of $P(x)$ 
		- $= x^6+x^4+x^3+x^2+x+1 = 01011111 = 0x5F$ 
	- Do for others to get $5F + 17 + 4C + 25$ 
		![](Pasted%20image%2020260225033000.png)
- The matrix was chosen such that this is invertible under decryption and that it diffuses maximally(4 col bytes affect 1 output byte, 4 times there fore if one input byte changes, all 4 output bytes change, across the whole matrix, achieving maximal diffusion)
	- This transformation is **linear**

### Key Schedule
- AES' block size is 128 bits and therefore each round needa 128 bit round key
	- Same even when using AES 192/256
- Cascade of XORs allows bit changes and key bits to propagate through the block, with S-box for confusion and rotations for diffusion in the key schedule. 
- For 10 roudns, need 11 round keys(key whitening at beginning) 11 x 128 = 1408 bits
	1. Split the key into bytes $(0-15)$ and group into 4 words of 4 bytes $W[0], W[1], W[2], W[3]$ 
	2. Generate remaining words, we need 44 total and we have 4 thus far
		- For AES-128 if $i$ is not a multiple of $4$:
			- $W[i] = W[i-4] \oplus W[i-1]$ 
		- If $i$ is a multiple of 4 then:
			- $W[i] = W[i-4] \oplus g(W[i-1])$ 
			- Where $g$ rotates a 4 byte word left once, then applies the AES s-box to each byte. Then XOR the first byte with a round constant $rcon_i$.
- ![](Pasted%20image%2020260225040718.png)


### Implementation
- All additions and subtractions are XOR, where elements are polynomials with coefficients in $GF(2^8)$ with coefficients in $GF(2)$, adding two polynomials means adding coefficients under $mod 2$ which is just XOR
	- Subtraction is the same as addition, and is also just XOR.
- Multiply by 01 has no effect in mix columns fixed matrix. 
- Multiply by 02(x) is a simple left shift followed by a modular reduction if the result is not in GF(2^8)
#### Inverses
- Encryption uses MixColumns with the matrices with constants 01, 02, 03 which are just shift once. 
- Decruption uses the constants 09, 11, 13, 14. 
- These are much harder. 
??? Cover???

#### More detail
- AES is quick in software, and very fast in hardware too
- Built into hardware; CPU instructions/hardware acceleration features such as AES-NI make galois field arithmetic possible directly on CPU
- Much of algorithm can be converted into lookup tables rather than computed live
	- TTrade-off
- Cache-timing and other side channel attacks e.g., rowhammer possible if the algorithm is implemented naively. 
	- E.g., if do S-box via table input byte, then the memory address accessed depends on state-dependent data, CPU cache and TLB misses etc make some accesses faster than others
	- Can observe this timing and infer which table entries were used to partially uncover info about input, potentially key/plaintexxt info
	- Want implementation to be in constant time, same instruction pattern regardless of key/plaintext so it is equally as fast for given input combination compared to another. 