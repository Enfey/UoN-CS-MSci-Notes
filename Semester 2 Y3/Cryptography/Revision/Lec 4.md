## Block ciphers
- One of the two main classes of symmetric encryption ciphers
- Operates on **fixed-size blocks of plaintext** and transforms each block into a ciphertext block of the same size, using a secret key.
- More common than stream ciphers.
	- Versatile primitives; used for other cryptographic methods e.g., MAC, hash constructions. 
- For a given key $k$ the encryption function: 
	$$E(k) : \{0,1\} \to \{0,1\}^n$$
	Must be **deterministic** and **invertible** 

- Block ciphers are combined with **modes of operation** to securely encrypt data of variable length.
- Doesn't use the same key all the time - depends on rounds and the mode of operation


### Pseudorandom permutations
- A **block cipher** is meant to behave like a **pseudorandom permutation** over $\{0, 1\}^n$ 
- This is a function that cannot be distinguished from a random **permutation** 
	- The output size equals the input size, and this is bijective, whereas CSPRNGs expand a short seed into a pseudrandom keystream to be computationally indgistinguihable from true randomness. 
- Maps a set of values $\{0, 1\}^n \times \{0,1\}^n \to \{0, 1\}^n$ 
	- For any key $k$ this function $F$ is a bijection(every plaintext has exactly one ciphertext, and every ciphertext comes from exactly one plaintext)
		- Otherwise decryption would be impossible
		- And if some plaintexts collide or some ciphertexts not possible, instantly gives away info. 
	- There is an efficient algorithm to calculate $F(m, k) = c$ for all keys and all messages.
- This is what we attempt to approximate, rather than a CSPRNG predicated on a small seed key. 


### Terminology
- **Confusion**
	- *Obscure the relationship between the **plaintext**, **key** and **ciphertext***
		- Confusion is often achieved in block ciphers via things like substitution tables and often involves injecting non-linearity
		- Things like word-wise adding in ChaCha20, the operations in the quarter round to permute bits etc.
- **Diffusion**
	- *Spread the influence of each input bit across many output biots*
		- Usually achieved via permutation e.g., swapping or otherwise mixing bits or bytes.
- Ciphers which repeatedly apply these ideas are called **product ciphers**
	- Apply rounds of subtitution and permutation sequentially to yield a ciphertext $c$ 

## Feistel Networks
- The feistel network is introduced, general construction method for block ciphers
- Uses round function, takes 2 inputs $L, R$, and a key $k$ and returns output same size as the data block
- In each round, the round function runs on half of the data to be encrypted and the round key, and the output of the round function is XORed with the other half
	![](Pasted%20image%2020260521203303.png)
- This repeats a fixed number of times, with keys $k$ different for each round
- The last round performs a final swap
- A major advantage compared to say SP networks is that the entire operation is guaranteed to be invertible even if the round function is **non-invertible**
	- Can be arbitrarily complicated. 
- Encryption and decryption are very similar; just reverse the key schedule.


### Feistel Round
- During each round, only **half of the block is encrypted**
- Take subkey $k$ and $R_i$ to produce $f(k_i, R_i)$ 
- This output is $XORed$ with $L_i$ yielding $L \oplus f(k_i, R_i)$ 
- The function f should behave as a pseudorandom function:
	$F_k : \{0,1\}^n \to \{0,1\}^m$ 
	Takes key and input and yields block size (where n is k+b)

### Feistel cipher encryption
- Start with $L_i, R_i$ 
- $L_i = R_i$, $R_i = L_i \oplus f(k, R_i)$ 
- $L_i = L_i \oplus f(k, R_i)$, $R_i = R_i \oplus f(L_i \oplus f(k, R_i), k+1)$ 
- Etc
- Final round does a final swap

### Feistel cipher decryption
-  Start with $L_i = R_i \oplus f(L_i \oplus f(R_i, k_i), k_{i+1}), R_{i} = L_i \oplus f(R_i, k_i)$  
	- After the initial function applied to the RHS, would XOR with LHS, but would yield $R_i$ because XOR is its own inverse
	- Thus, it does not matter whether the round function is invertible because thw wider structure it exists in is.

### Feistel network design
- 1 or 2 rounds not sufficient to yield cipher
	- 1 round still has plaintext
	- 2 round is further distinguishable as both have been confused and diffused.
- Proved that if round function $f$ applied to one of the two halves behaves like a secure **pseudorandom function** then a 3-round feistel gives you a $PRF$, meaning it stays psuedorandom even if an adversay can query both encryption and decryption
	- A cipher is a **PRP** if an attacker who can query only encryption cannot tell whether they are talking to block cipher or random permutation
	- A cipher is a **strong PRP** if this is the same for encryption and decryption.
- **Balanced feistel network**
	- L and R are equal sizes
- **Unbalanced feistel network**
	- They are not
- **Skipjack**
	- Block cipher using feistel like structure but heavily exceeds minimum 4 rounds, remains secure under very strong attack models
- **OAEP**
	- Padding/encoding scheme for RSA
	- Structurally similar to unbalanced Feistel network, mixes short random seed and long message block
	- Feistel ideas useful outside of symmetric encryption.

## Data Encryption Standard
- Symmetric key algorithm for encryption.
- Block cipher
- 56 bit key length (8 parity bits, makes 64, one in each byte for error detection in keygen, distribution, and storage)
- Block size of 64 bits
- 16 round feistel network
	- Guarantees invertbility
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
	- The first and last bit form the row bits.
	- The middle bits form the column bits. 
	![](Pasted%20image%2020260211002025.png)
	![](Pasted%20image%2020260211002043.png)
	ROW, THEN COL BITS

##### S-box design
- Need to be highly non-linear, they introduce non-linearity and confusion after XOR to prevent breaking the cipher down to a system of linear equations; permitting retrieveal of internal state and ability to discover key.
- Key design principles:
	1. **No Output bit should be too close to a linear combination of input bits**
	2. **1 bit change in input should lead to at least 2 bit change in output**
		Local avalanche
	3. **If only the middle 4 bits change each output must occur exactly once**
		- Necessitates 0-15; if the row is fixed, make the output equally likely. 
	4. **If the first two bits are different but the last two are identical, the output must differ**
	5. If two inputs differ by delta, their outputs should rarely differ by the same delta
	6. A collision is only possible for yeahn idk

#### Permutation
- ![](Pasted%20image%2020260211004224.png)
- Given the feistel equations $L_i = R_i$, $R_i = L_i \oplus f(R_i, k)$ the permutation $P$ is inside $F$
- The 32 bit output from $S-boxes$ are rearranged according to a fixed permutation, the $P-box$ 
- This is designed such that the output of the independent S-boxes are spread across multiple different S-boxes in the next round, and that they do not stay isolated, enabling both confusion and diffusion. 