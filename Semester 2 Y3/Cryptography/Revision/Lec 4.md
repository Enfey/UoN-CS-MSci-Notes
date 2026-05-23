## Block ciphers
- One of the t<mark style="background: #FFF3A3A6;">wo main classes of symmetric encryption ciphers</mark>
- Operates on **fixed-size blocks of plaintext** and transforms each block into a ciphertext block of the same size, using a secret key.
- <mark style="background: #FFF3A3A6;">More common than stream ciphers</mark>.
	- Versatile primitives; used for other cryptographic methods e.g., MAC, hash constructions. 
- For a given key $k$ the encryption function: 
	$$E(k) : \{0,1\} \to \{0,1\}^n$$
	Must be **deterministic** and **invertible** 

- <mark style="background: #FFF3A3A6;">Block ciphers are combined</mark> with **modes of operation** to s<mark style="background: #FFF3A3A6;">ecurely encrypt data of variable length.</mark>
- Doesn't use the same key all the time - depends on rounds and the mode of operation


### Pseudorandom permutations
- A **block cipher** is meant to behave like a **pseudorandom permutation** over $\{0, 1\}^n$ 
- This is a <mark style="background: #FFF3A3A6;">function that cannot be distinguished from a random</mark> **permutation** 
	- <mark style="background: #FFF3A3A6;">The output size equals the input size</mark>, and <mark style="background: #FFF3A3A6;">this is bijective</mark>, whereas CSPRNGs expand a short seed into a pseudrandom keystream to be computationally indgistinguihable from true randomness. 
- Maps a set of values $\{0, 1\}^n \times \{0,1\}^n \to \{0, 1\}^n$ 
	- For any key $k$ this function $F$ is a bijection(e<mark style="background: #FFF3A3A6;">very plaintext has exactly one ciphertext</mark>, and <mark style="background: #FFF3A3A6;">every ciphertext comes from exactly one plaintext</mark>)
		- <mark style="background: #FFF3A3A6;">Otherwise decryption would be impossible</mark>
		- <mark style="background: #FFF3A3A6;">And if some plaintexts collide</mark> or <mark style="background: #FFF3A3A6;">some ciphertexts not possible</mark>, instantly <mark style="background: #FFF3A3A6;">gives away info. </mark>
	- There is an efficient algorithm to calculate $F(m, k) = c$ for all keys and all messages.
- <mark style="background: #FFF3A3A6;">This is what we attempt to approximate</mark>, rather than a CSPRNG predicated on a small seed key. 


### Terminology
- **Confusion**
	- <mark style="background: #FFF3A3A6;">*Obscure the relationship</mark> between the **plaintext**, **key** and **ciphertext***
		- <mark style="background: #FFF3A3A6;">Confusion is often achieved in block ciphers via things like substitution tables</mark> and often<mark style="background: #FFF3A3A6;"> involves injecting non-linearity</mark>
		- Things like <mark style="background: #FFF3A3A6;">word-wise adding </mark>in ChaCha20, the <mark style="background: #FFF3A3A6;">operations in the quarter round</mark> to permute bits etc.
- **Diffusion**
	- *Spread the influence of each input bit across many output biots*
		- <mark style="background: #FFF3A3A6;">Usually achieved via permutation</mark> e.g.,<mark style="background: #FFF3A3A6;"> swapping or otherwise</mark> <mark style="background: #FFF3A3A6;">mixing bits or bytes.</mark>
- Ciphers which repeatedly apply these ideas are called **product ciphers**
	- Apply rounds of subtitution and permutation sequentially to yield a ciphertext $c$ 

## Feistel Networks
- The feistel network is introduced, g<mark style="background: #FFF3A3A6;">eneral construction method for block ciphers</mark>
- Uses <mark style="background: #FFF3A3A6;">round function</mark>, takes <mark style="background: #FFF3A3A6;">2 inputs</mark> $L, R$, and a <mark style="background: #FFF3A3A6;">key</mark> $k$ and <mark style="background: #FFF3A3A6;">returns output</mark> <mark style="background: #FFF3A3A6;">same size as the data block</mark>
- In<mark style="background: #FFF3A3A6;"> each round,</mark> the<mark style="background: #FFF3A3A6;"> round function</mark> runs on <mark style="background: #FFF3A3A6;">half of the data to be encrypted </mark>and the <mark style="background: #FFF3A3A6;">round key</mark>, and the <mark style="background: #FFF3A3A6;">output of the round function</mark> is <mark style="background: #FFF3A3A6;">XORed with the other half</mark>
	![](Pasted%20image%2020260521203303.png)
- This <mark style="background: #FFF3A3A6;">repeats a fixed number of times</mark>, with keys $k$ different for each round
- <mark style="background: #FFF3A3A6;">The last round performs a final swa</mark>p
- A <mark style="background: #FFF3A3A6;">major advantage</mark> compared to say SP networks is that the<mark style="background: #FFF3A3A6;"> entire operation</mark> is <mark style="background: #FFF3A3A6;">guaranteed to be invertible</mark> <mark style="background: #FFF3A3A6;">even if the round function</mark> is **non-invertible**
	- Can be arbitrarily complicated. 
- <mark style="background: #FFF3A3A6;">Encryption and decryption</mark> are very similar; just<mark style="background: #FFF3A3A6;"> reverse the key schedule.</mark>


### Feistel Round
- During each round, only **half of the block is encrypted**
- Take subkey $k$ and $R_i$ to produce $f(k_i, R_i)$ 
- This output is $XORed$ with $L_i$ yielding $L \oplus f(k_i, R_i)$ 
- <mark style="background: #FFF3A3A6;">The function f should behave as a pseudorandom</mark> function:
	$F_k : \{0,1\}^n \to \{0,1\}^m$ 
	<mark style="background: #FFF3A3A6;">Takes key and input</mark> and <mark style="background: #FFF3A3A6;">yields block size</mark> (where n is k+b)

### Feistel cipher encryption
- Start with $L_i, R_i$ 
- $L_i = R_i$, $R_i = L_i \oplus f(k, R_i)$ 
- $L_i = L_i \oplus f(k, R_i)$, $R_i = R_i \oplus f(L_i \oplus f(k, R_i), k+1)$ 
- Etc
- Final round does a final swap

### Feistel cipher decryption
-  Start with $L_i = R_i \oplus f(L_i \oplus f(R_i, k_i), k_{i+1}), R_{i} = L_i \oplus f(R_i, k_i)$  
	- After the initial function applied to the RHS, <mark style="background: #FFF3A3A6;">would XOR with LHS</mark>, but <mark style="background: #FFF3A3A6;">would yield</mark> $R_i$ because <mark style="background: #FFF3A3A6;">XOR is its own inverse</mark>
	- <mark style="background: #FFF3A3A6;">Thus,</mark> it <mark style="background: #FFF3A3A6;">does not matter whether the round function is invertible</mark> because <mark style="background: #FFF3A3A6;">thw wider structure it exists in is.</mark>

### Feistel network design
- 1<mark style="background: #FFF3A3A6;"> or 2 rounds not sufficient to yield cipher</mark>
	- <mark style="background: #FFF3A3A6;">1 round still has plaintext</mark>
	- <mark style="background: #FFF3A3A6;">2 round is further distinguishable as both have been confused and diffused.</mark>
- <mark style="background: #FFF3A3A6;">Proved that if round function</mark> $f$ <mark style="background: #FFF3A3A6;">applied to one of the two halves behaves like</mark> a secure **pseudorandom function** then a<mark style="background: #FFF3A3A6;"> 3-round feistel</mark> gives you a $PRF$, <mark style="background: #FFF3A3A6;">meaning it stays psuedorandom</mark> <mark style="background: #FFF3A3A6;">even if an adversay</mark> can <mark style="background: #FFF3A3A6;">query both encryption</mark> and <mark style="background: #FFF3A3A6;">decryption</mark>
	- A cipher is a **PRP** if an attacker who can <mark style="background: #FFF3A3A6;">query</mark> <mark style="background: #FFF3A3A6;">only encryption</mark> <mark style="background: #FFF3A3A6;">cannot tell</mark> whether they are<mark style="background: #FFF3A3A6;"> talking to block cipher</mark> or <mark style="background: #FFF3A3A6;">random permutation</mark>
	- <mark style="background: #FFF3A3A6;">A cipher</mark> is a **strong PRP** <mark style="background: #FFF3A3A6;">if this is the same</mark> for <mark style="background: #FFF3A3A6;">encryption and decryption.</mark>
- **Balanced feistel network**
	- L and R are equal sizes
- **Unbalanced feistel network**
	- They are not
- **Skipjack**
	-<mark style="background: #FFF3A3A6;"> Block cipher using feistel like structure but heavily exceeds minimum 4 rounds, remains secure under very strong attack models</mark>
- **OAEP**
	- Padding/encoding scheme for RSA
	- Structurally similar to unbalanced Feistel network, mixes short random seed and long message block
	- Feistel ideas useful outside of symmetric encryption.

## Data Encryption Standard
- <mark style="background: #FFF3A3A6;">Symmetric key algorithm for encryption</mark>.
- <mark style="background: #FFF3A3A6;">Block cipher</mark>
- 56 bit key length (8 parity bits, makes 64, one in each byte for error detection in keygen, distribution, and storage)
- Block size of 64 bits
<mark style="background: #FFF3A3A6;">- 16 round feistel network</mark>
	<mark style="background: #FFF3A3A6;">- Guarantees invertbility</mark>
	- Can reuse same round function for encryption and decryption.

### Overall structure 
- 16 stages of processing via Feistel network; rounds
- Initial and final permutation; $IP$ and $FP$
	- $IP$ undoes the action of $FP$, no cryptographic significance
	- Specify fixed bit rearrangement of plaintext so first 32 bits naturally loaded into register $L$ and the rest into $R$
- $IP$ divided into 32-bit halves and processed accoridngly
- Only difference in decryption is reversal in subkey application order/scheduling.


### PC-1 and transformations
- Discard parity first and foremost - PC-1
- Then rearrange to get remaining key bits to spread key bits across diff rounds
	- Split 56 bit key into $C$ and $D$ halves
	- Each round $C_i$ and $D_i$ are left rotated(right rotate for decryption)
	- Get distinct key per round, but the selection is deterministic
- As input to Feistel function, 48 bits selected, generally omit different bits each round depending on rotation history to prevent attacks that assume simple key progression. 
- 16 distinct 48 bit round keys $K_0...K_{15}$ every 1 bit change in master key diffuses across many transforms.

### The feistel function F
- The $F$ function operates on half a block and a subkey $k$
- Consists of 4 stages:
	1. **Expansion**
	2. **Key mixing**
	3. **Substitution**
	4. **Permutation**

#### Expansion
- DES needs 48 bits to be able to XOR the 48 bit round key $k$ with one of the 32 bit block halves
- Expansion cannot invent new information; reuse existing bits
- There is a fixed expansion lookup table whereby half of the input bits are connected to 2 of the output bits.. 
	![](Pasted%20image%2020260211001045.png)
	This adds diffusion; spreading 50% of input bits to 2 output bits. 

#### Key mixing
- The expanded 48 bit value is XORed with the round key to yield; $E(R_i) \oplus K_i$; E denotes the expansion function
- $XOR$ is linear under $GF(2)$ and so we injet non-linearity into our round function via the use of substitution

#### Substitution
- **Sboxes** map 6 bit input into 4 bit outputs
- There are 8 sboxes in total, where each is different
- The 48 bit input is split into 8 chunks of 6 bits with S-boxes applied to each constituent part to yield 32 bit output. 

##### S-box application
- Split the 6 bits into row bits and column bits
	- <mark style="background: #FFF3A3A6;">The first and last bit form the row bits.</mark>
	- <mark style="background: #FFF3A3A6;">The middle bits form the column bits. </mark>
	![](Pasted%20image%2020260211002025.png)
	![](Pasted%20image%2020260211002043.png)
	ROW, THEN COL BITS

##### S-box design
- <mark style="background: #FFF3A3A6;">Need to be highly non-linear</mark>, they introduce<mark style="background: #FFF3A3A6;"> non-linearity and confusion after XOR</mark> to <mark style="background: #FFF3A3A6;">prevent breaking the cipher down to a system of linear equations</mark>; permitting retrieveal of internal state and ability to discover key.
- Key design principles:
	1. **No Output bit should be too close to a linear combination of input bits**
	2. **1 bit change in input should lead to at least 2 bit change in output**
		Local avalanche
	3. **If only the middle 4 bits change each output must occur exactly once**
		- Necessitates 0-15; if the row is fixed, make the output equally likely. 
	4. **If the first two bits are different but the last two are identical, the output must differ**
	5. I<mark style="background: #FFF3A3A6;">f two inputs differ by delta, their outputs should rarely differ by the same delta</mark>
	6. A collision is only possible for yeahn idk

linear COMB, DELTA, MIDDLE BITS, LOCAL AVALANCHE. 

#### Permutation
- ![](Pasted%20image%2020260211004224.png)
- <mark style="background: #FFF3A3A6;">Given the feistel equations $L_i = R_i$, $R_i = L_i \oplus f(R_i, k)$ the permutation $P$ is inside $F$</mark>
- <mark style="background: #FFF3A3A6;">The 32 bit output </mark>from $S-boxes$ are<mark style="background: #FFF3A3A6;"> rearranged according to a fixed permutation</mark>, the $P-box$ 
- This is<mark style="background: #FFF3A3A6;"> designed such that the output of the independent S-boxes are spread across multiple different S-boxes in the next round, and that they do not stay isolated,</mark> enabling both confusion and diffusion. 