## Advanced Encryption Standard
- Specification for the encryption of data, superseded DES as standard in 2002. 
- It is a variant of the **Rijndael** block cipher
- Rijndael is an SP-network with 128-bit block size, and a key length of 128, 192, or 256-bits.
	- Recall that these are block ciphers that combine multiple rounds of substitution (S-boxes) and permutation (p-boxes) mixed with subkeys/round keys, to achieve confusion and diffusion respectively. 
- 10, 12, or 14 rounds. 
- Unlike DES, AES transforms the whole block each round.
	![](Pasted%20image%2020260224225732.png)
	Note that there is more rounds and the key schedule changes depending on the key size. 

### AES Rounds structures
- **Key Expansion**
	- Each round receives its own key, generated from the master key. 
	- 16 byte key for AES-128, word = 4 bytes, so 4 words per round key. 
		- Need to generate 44 words of keys from the master key for 10 round keys (starting from 0).
- Uses rounds of 4 layers, and a final round of 3 layers
	![](Pasted%20image%2020260225005142.png)
	- As can see, initially have a subkey that is XOR'd with the plaintext. 
	- Then have 3 operations, and then XOR the next round key in again. 
	- Do this for every round except the last one where don't do a mixed columns. 

#### Representation and manipulation
- Bytes are represented as a $4 \times 4$ block called the **state**
	![](Pasted%20image%2020260225005424.png)
	Implemented as a 2D array. 
- For every byte in state, visit SubBytes lookup table, which takes as input 8 bits and returns 8 bits as output, and return a different byte.
- For each row in the state, the ShiftRow transformation performs a left-rotation on each row whose index $>1$, e.g., row $2$ has a 1 byte left rotation, row $3$ has a $2$ byte rotation etc. 
	![](Pasted%20image%2020260225010130.png)
- For each column in the state, the MixColumns applies a fixed transformation to achieve diffusion, taking 4 byte chunks and mixing them together linearly.


### S-box
- The AES S-box is based around the multiplicative inverse of 8-bit values in $GF(2^8)$ 
	![](Pasted%20image%2020260225012705.png)
	Split into 2, interpret byte as polynomial, inverse is initial S-box entry.
- We interpret bytes as a polynomial in $GF(2^8)$ and compute the multiplicative inverse under Rijndahl's irreducible modular reduction (except for 0) for every byte from 0 to 255.
	- $A_i \cdot A_i^{-1} \equiv 1 \ mod (P(x))$ 
	- Decent so far in creating a strongly non-linear mapping(Field inversion is highly non-linear it involves polynomial division and high-degree algebraic expressions), but 0 stays as a fixed point which we want to avoid. 
	- We say that:$$
B'_i =
\begin{cases}
0 & \text{if } i = 0 \\
A_i^{-1} & \text{if } i > 0
\end{cases}
$$
- The inverses $B'_i$ then undergo an **affine transformation** to produce the final $S-box$, destroying any remaining mathematical structure. 
	- An affine transformation is a linear transformation plus a linear vector.  ![](Pasted%20image%2020260225012858.png)
		- They simply just fixed the matrix and incremented the values until they got desirable $S-box$.
#### S-box properties
- The s-box is bijective and is therefore an invertible 1:1 mapping. 
- It maintains no fixed points via the affine transformation and rounds, i.e., no $A_i$ for which $S(A_i) = A_i$ 
- No inverse fixed points, that is $S(A_i) \oplus A_i = FF$, aka $A_i$ should not be its own bitwise complement (1's reveal the difference) under the $S-box$.
	- Example:
		- $x = 01010101$ 
		- $FF = 11111111$ 
		- We can rearrange $S(A_i) = A_i \oplus FF$, so if the $S-box$ did $S(01010101) = 10101010$ that would be an inverse fixed point, changed in every position, simple algebraic structure.
- **It is difficult to perform linear cryptanalysis.** 
	- Over $GF(2)$ a linear combination just means $XOR$ (which is linear over bits).
	- For a given equation $x_1 \oplus x_2 = y_1$ we say that the XOR of two input bits equals one output bit.
	- We check for all 256 possible inputs to the $S-box$ how often this equation is true, if the S-box behaves randomly, then it should be true exactly 50% of the time. 
	- There is no way to represent input bits and output bits as linear combinations that reveal bias in the s-box that can be solved;  no linear equation is strongly predictive of the S-box output
	- Minimises the largest bias, if do for one S-box, the chance of being able to perform linear cryptanalysis on the whole cipher is going to be extremely difficult/impossible.
- **Minimisation of the largest non-trivial value in the EXOR table**
	- We say there is no likely predictable output difference from some input difference $Δ$.
	- The largest likelihood of some predictable change in the ouput in one S-box in one of 10 rounds is $2^{-6}$ 
### AES Diffusion
- Diffusion in AES consists of two layers
	1. Shift rows
	2. Mix columns
- Shift rows simply performs a left-rotation on each row whose index $>1$, e.g., row $2$ has a 1 byte left rotation, row $3$ has a $2$ byte rotation etc. 
	![](Pasted%20image%2020260225010130.png)
	- Weak on its own because localises diffusion to the row, and does not affect many bytes
	- However, it is needed by MixColumns to create global diffusion across the state by ensuring that columns do not stay independent. 

#### MixColumns
- Performs a linear mixing of bytes within each column.
- It does this by representing those bytes as polynomials in Galois, $GF(2^8)$ and then performing matrix-vector multiplication to yield a column vector denoting the output column(s).
- All of the input bytes in a column influence all of the output bytes, as they are multiplied in each step as can be seen below to yield individual vector components. 
	![](Pasted%20image%2020260225022029.png)
- For example:
	![](Pasted%20image%2020260225033847.png)
	- $C_0 = 02 \cdot A2 + 03 \cdot 0D + 01 \cdot 4C + 01 \cdot 25$
		These bytes are polynomials, and thus addition on them essentially resolves to XOR in accordance with the prime subfield $GF(2)$
	- $02 \cdot A2=$
		- $x \cdot (x^7 + x^5 + x)$ 
			- 02 = 00000010
			- A2 = 10100010
		- $= x^8 + x^6 + x^2$ 
			- Must now do polynomial long division according to $x$as exceeds $GF(2^8)$ 
		- $=x^6 + x^2 + x^4 + x^3 + x + 1$ 
			- This was acquired via a shortcut, where x^8 was substituted in accordance with the rearrangement of $P(x)$ 
		- $= x^6+x^4+x^3+x^2+x+1 = 01011111 = 0x5F$ 
	- Do for others to get $5F + 17 + 4C + 25$ 
		- ![](Pasted%20image%2020260225033000.png)
		Where the addition is under XOR, the cancels make perfect sense. 
- The matrix was chosen such that this is invertible under decryption and that it diffuses maximally. By nature of matrix-vector multiplication, if one input byte changes, all 4 output bytes change, essentially achieving maximal diffusion. 
	- Do note that this transformation is linear. 
### Key Schedule
- AES's block size = 128 bits, therefore each round needs a 128 bit round key
	- This is the same regardless of whether using AES-128, AES-192, or AES-256.
- Cascade of XORs which allows bit changes and key bits to propagate through the block, with S-box for confusion between keys and rotations for diffusion.
- For 10 rounds, need 11 round keys, 11 x 128 = 1408 bits.
	1. Split the key into bytes $(0-15)$ and group into 4 words of 4 bytes $W[0], W[1], W[2], W[3]$ 
	2. Generate remaining words, we need $44 (4 \times 11)$ total
		- For AES-128 if $i$ is not a multiple of 4:
			- $W[i] = W[i-4] \oplus W[i-1]$ 
		- If $i$ is a multiple of $4$ (every 4th word, that is)
			- $W[i] = W[i-4] \oplus g(W[i-1])$ 
			- Every 4th word uses the function $g()$ 
				- The function $g()$ rotates a 4-byte word left once, and then applies the AES S-box to each byte. Then XOR the first byte with a round constant $rcon_i$, an $8-bit$ value.
		![](Pasted%20image%2020260225040718.png)
		Note the arrow after the XOR which satisfies the above rule.


### Implementation
1. All additions and subtractions are XOR (under $GF(2^8)$ where elements are polynomials with coefficients in $GF(2)$, adding two polynomials means adding coefficients under mod $2$, which is just XOR. Subtraction is the same as addition in mod 2, so subtraction is also just XOR).
2. Multiply by 01 has no effect in MixColumns.
3. Multiply by 02(x) is simply a left shift followed by modular reduction if the result is not in $GF(2^8)$
	- If the original $x^7$ bit was set, then we must XOR with $0x1B$ 
		- This is the trick described earlier in mix columns to remove $x^8$ in the polynomial, but represented as binary, with XOR to wrap back round. 
			![](Pasted%20image%2020260225041955.png)
- Multiply by 03 (x+1) is xtime(a) ^ a
	![](Pasted%20image%2020260225043427.png)
	![](Pasted%20image%2020260225043056.png)
#### Inverses
- Encryption MixColumns uses the matrix with constants 01, 02, 03, inverse MixColumns uses the constants 00, 11, 13, 14.
- These are harder as they are not just shift once. 
	![](Pasted%20image%2020260225043359.png)
	Unfinished.
#### More detail
- AES is extremely quick in software, and also very fast in hardware too. 
- CPU instructions in AES-NI make AES much faster.
	- Hardware acceleration feature that boots encryption and decryption speeds, offers galois field arithmetic etc directly on the CPU. 
- Much of the algorithm can be converted into series of lookup tables
	- Trade-off between speed or space. 
- Cache-timing and other side channel attacks possible if the algorithm is implemented naively, as in directly from the RFC. 
	- E.g., if do S-box via table[input_byte] then the memory address accessed depends on the state-dependent data, and CPU cache makes some accesses faster than others. 
	- Attacker could observe cache behaviour and infer which table entries were used, leaking info about the key. 
	- Want implementation to be in constant time, that is, the same instruction pattern regardless of key/plaintext, and not having secret dependent branches or memory indexing that could leak key information. 