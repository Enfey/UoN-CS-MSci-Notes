## Advanced Encryption Standard
- <mark style="background: #FFF3A3A6;">Specification for the encryption of data</mark> <mark style="background: #FFF3A3A6;">following DES</mark>
- Variant of the **rijndael** block cipher
	- <mark style="background: #FFF3A3A6;">SP-network with 128-bit block size</mark>, key length of 128, 192, 256-bits
	- <mark style="background: #FFF3A3A6;">Recall these are block ciphers</mark> whose r<mark style="background: #FFF3A3A6;">ounds consist of substitution</mark> via s-boxes and <mark style="background: #FFF3A3A6;">permutation mixed</mark> with s<mark style="background: #FFF3A3A6;">ubkeys to achieve confusion</mark>/<mark style="background: #FFF3A3A6;">diffusion</mark>, respectively. 
- 10,12,14 rounds
- Unlike DES, AES transforms the whole block each round
	![](Pasted%20image%2020260224225732.png)
	Note there is more rounds, and the key schedule changes depending on the key size. 


### AES rounds structure
- **Key expansion**
	- <mark style="background: #FFF3A3A6;">Each round</mark> <mark style="background: #FFF3A3A6;">receives own key</mark> <mark style="background: #FFF3A3A6;">generated from master key</mark>
	- <mark style="background: #FFF3A3A6;">16 byte key</mark> for AES-128, word = 4 bytes, thus, <mark style="background: #FFF3A3A6;">4 words per round key</mark>. 
		- Need to <mark style="background: #FFF3A3A6;">generate then</mark>, 4<mark style="background: #FFF3A3A6;">4 words from the master key for 12 round keys</mark>
- Uses <mark style="background: #FFF3A3A6;">rounds of 4 layers</mark>, and a <mark style="background: #FFF3A3A6;">final round of 3 layers</mark>
	![](Pasted%20image%2020260225005142.png)
	As can see, have a subkey that is initially mixed into the plaintext, then have 3 operations, then XOR the next round key in again. 
	This continues for all rounds, except the last one, where this is no mixcolumns. 

### Representation and manipulation
- Bytes are represents as a 4x4 block called the **state**
	![](Pasted%20image%2020260225005424.png)
- For every byte in state, <mark style="background: #FFF3A3A6;">visit SubBytes lookup table</mark>, which <mark style="background: #FFF3A3A6;">takes as input 8 bits</mark>, and <mark style="background: #FFF3A3A6;">returns 8 bits as output,</mark> <mark style="background: #FFF3A3A6;">returning a different byte</mark>
- <mark style="background: #FFF3A3A6;">For each row in the state</mark>, the<mark style="background: #FFF3A3A6;"> ShiftRow transformation performs a left rotation</mark> on <mark style="background: #FFF3A3A6;">each row whose index</mark> > 1 e.g., <mark style="background: #FFF3A3A6;">row 2 has 1 byte left rotation</mark>, r<mark style="background: #FFF3A3A6;">ow 3 has a 2 byte rotation etc. </mark>
	![](Pasted%20image%2020260225010130.png)
-<mark style="background: #FFF3A3A6;"> For each column in the state</mark>, the<mark style="background: #FFF3A3A6;"> MixCoumns applies a fixed transformatio</mark>n to achieve diffusion, taking 4 byte chunks and mixing them together linearly.

### S-box
- <mark style="background: #FFF3A3A6;">The AES s-box </mark>is <mark style="background: #FFF3A3A6;">based around the multiplicative inverse</mark> of $8$ <mark style="background: #FFF3A3A6;">bit values</mark> in $GF(2^8)$ 
	![](Pasted%20image%2020260225012705.png)
	Split byte into two, interpret as polynomial, inverse is initial S-box entry. 
- <mark style="background: #FFF3A3A6;">We interpret bytes as a polynomial</mark> in $GF(2^8)$ and com<mark style="background: #FFF3A3A6;">pute the multiplicative inverse</mark> via EAA for every byte from 0 to 255. 
	$A_i \cdot A_i^{-1} \equiv 1 \ mod (P(x))$ y<mark style="background: #FFF3A3A6;">ielding the neutral element of the prime-extension field under this irreducible polynomial.</mark>
	- Decent in creating a strongly non-linear mapping; <mark style="background: #FFF3A3A6;">field inversion is non-linear</mark> in that it <mark style="background: #FFF3A3A6;">involves polynomial division, </mark>but <mark style="background: #FFF3A3A6;">0 stays as a fixed point</mark> as it has <mark style="background: #FFF3A3A6;">no multiplicative inverse, </mark>which we <mark style="background: #FFF3A3A6;">want to avoid</mark>
- We say that:$$
B'_i =
\begin{cases}
0 & \text{if } i = 0 \\
A_i^{-1} & \text{if } i > 0
\end{cases}
$$
- The inverses $B'_i$ undergo an <mark style="background: #FFF3A3A6;">**affine transformation**</mark> to produce the final $S-box$ <mark style="background: #FFF3A3A6;">destroying any remaining mathematical structure</mark>
	- An affine transformation is a linear transformation plus a linear vector. 
		![](Pasted%20image%2020260225012858.png)
	<mark style="background: #FFF3A3A6;">Matrix is fixed, and incremented the values until they got the desirable</mark> $S-box$ such that <mark style="background: #FFF3A3A6;">$0$ no longer maps to zero which would not be strong against differential/linear cryptanalysis as it is a fixed point </mark>under the prime binary extension field as it has no modular multiplicative inverse.


### S-box porperties
- The $s-box$ is <mark style="background: #FFF3A3A6;">bijective</mark>, and is <mark style="background: #FFF3A3A6;">therefore an invertible 1:1 mapping </mark>for a given byte $b$ 
- It <mark style="background: #FFF3A3A6;">maintains no fixed points</mark> via the affine transformation, there is no $A_i$ for which $S(A_i) = A_i$ 
- <mark style="background: #FFF3A3A6;">There is no inverse fixed points</mark>, that is $S(A_i) \oplus A_i = FF$<mark style="background: #FFF3A3A6;"> should not be its own bitwise complemen</mark>t(remember to think of it as polynomial and plus the coefficients). The 1s in between reveal the difference
	- Simple algebraic structure. If even a few inputs behave like this, there would be linearity to exploit within the S-box, whose design is meant to be entirely non-linear and defeat linear cryptanalysis
- **It is difficult to perform linear cryptanalysis**
	- Over $GF(2)$ a linear combination just means $XOR$ which is a linear function over bits. 
	- For a given equation $x_1 \oplus x_2  = y_1$ we say that the XOR of two input bits equals one output bit
		![](Pasted%20image%2020260522012429.png)
	- We check for all possible 256 inputs to the $S-box$ how often this equation is true, if the $S-box$ behaves randomly, it should be 50% of the time. 
	- <mark style="background: #FFF3A3A6;">No way to represent the input bits and output bits as linear combinations that reveal bias in the s-box; cannot predict input given its output. </mark>
	- Minimises the largest bias as if you do this for one s-bix the ability to perform linear cryptanalysis on the entire cipher will be extremely difficult/impossible.
- **DIFFERENTIAL PREVENTION**
	- <mark style="background: #FFF3A3A6;">We say there is no likely predictable output difference from some input difference</mark> $\Delta$ and is <mark style="background: #FFF3A3A6;">thus resistant to differential cryptanalysis;</mark> <mark style="background: #FFF3A3A6;">all possibilities are equally likely for each given byte</mark>; 4 bit portion, and it is nonp-linear. 


### AES Diffusion
- Diffusion in AES consists of two layers:
	1. <mark style="background: #FFF3A3A6;">**Shift rows**</mark>
	2. <mark style="background: #FFF3A3A6;">**Mix columns**</mark>
- <mark style="background: #FFF3A3A6;">Shift rows simply performs left rotation on each row whose index exceeds 1</mark> e.g., left rotate 1 for row 2 etc.
	![](Pasted%20image%2020260225010130.png)
- <mark style="background: #FFF3A3A6;">Weak on its own</mark> because it<mark style="background: #FFF3A3A6;"> localises the diffusion to the row,</mark> and does not affect bytes
- However, it is <mark style="background: #FFF3A3A6;">needed by mixcolumns</mark> to <mark style="background: #FFF3A3A6;">create global diffusion </mark><mark style="background: #FFF3A3A6;">with respect to the s-box output;</mark> ensures columns do not stay independent. 

### MixColumns
- <mark style="background: #FFF3A3A6;">Performs a **linear mixing** of bytes within each column</mark>
- It does this by <mark style="background: #FFF3A3A6;">representing those bytes as polynomials</mark> in Galois $GF(2^8)$ then <mark style="background: #FFF3A3A6;">performing matrix-vector multiplication</mark> to yield a column vector denoting all the output columns
- <mark style="background: #FFF3A3A6;">All of the input bytes</mark> in a<mark style="background: #FFF3A3A6;"> column influence</mark> <mark style="background: #FFF3A3A6;">all of the output bytes</mark>, as<mark style="background: #FFF3A3A6;"> they are all multiplied in each step</mark> to y<mark style="background: #FFF3A3A6;">ield individual vector components. </mark>
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
- The matrix was chosen such that<mark style="background: #FFF3A3A6;"> this is invertible under decryption and that it diffuses maximally</mark>(4 col bytes affect 1 output byte, 4 times there fore if one input byte changes, <mark style="background: #FFF3A3A6;">all 4 output bytes change,</mark> across the whole matrix, achieving maximal diffusion)
	- T<mark style="background: #FFF3A3A6;">his transformation</mark> is<mark style="background: #FFF3A3A6;"> **linear</mark>**
REPRESENT BYTES IN COLS AS POLYNOMIALS, MULTIPLY AGAINST ROW, AND ADD>
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