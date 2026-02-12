## Block Ciphers
- One of 2 main classes of symmetric encryption schemes (the other being stream ciphers)
- A block cipher operates on **fixed-size blocks of plaintext** and transforms each block into a ciphertext block of the same size using a secret key.
- These are generally more common than stream ciphers. 
	- This is because they are versatile primitives e.g., can be used for message authentication codes, hash constructions etc.
- For a given key $k$ the encryption function:$$E_{k} : \{0, 1\}^n \to \{0, 1\}^n$$
	Must be **deterministic**, and **invertible.**
- Unlike stream ciphers, block ciphers do not encrypt arbitrarily long messages, instead, they are combined with modes of operation (see security notes) to securely encrypt data of any length. 
- Technically doesn't use same key all the time, depends on **rounds** and mode of operation. 

## Pseudorandom Permutations
- A block cipher is meant to be behave like a **pseudorandom permutation** over $\{0, 1\}^n$ 
	- A **pseudorandom permutation** is a function that cannot be distinguished from a random permutation. 
- Maps a set of values $\{0, 1\}^n \times \{0, 1\}^n \to \{0, 1\}^n$
	- For any key $k$ this function $F$ is a **bijection** (every plaintext has exactly one ciphertext, and every ciphertext comes from exactly one plaintext). Zero collisions, otherwise decryption/inversion would be impossible. 
		- Non-bijectve leaks info e.g., if some ciphertexts are impossible or some plaintexts collide, immediately can deduce things.
	- There is an **efficient algorithm** to calculate $F(x)$ for all keys and all messages.
		- Needs to be usable.
- Unlike stream ciphers which approximate CSPRNGs by exploiting modular arithmetic under $XOR$, this is what we're trying to approximate.
	- This means they escape bijection as they don't emulate a function from blocks to blocks.
- Encryption is thus a psuedorandom permutation of all $2^n$ blocks.
## Terminology
- Claude Shannon introduced some important foundational principles to cryptography for the design of block ciphers
	- **Confusion**
		- *Obscure the relationship between the **plaintext**, **key**, and **ciphertext***
			- Confusion is often achieved via substitution operations e.g., lookup tables
	- **Diffusion**
		- *Spread the influence of each input bits across many output bits*
			- Diffusion is usually achieved via permutation e.g., swapping or otherwise mixing bits or bytes
- Shannon called a cipher that repeatedly combined these ideas into **rounds** **product ciphers**.
	![](Pasted%20image%2020260210210011.png)

## Feistel networks
- The feistel network is introduced as a general construction method for building block ciphers
- A feistel network uses a round function, which takes two inputs, a data block, and a key, and returns one output of the same size as the data block
- In each round, the round function is run on half of the data to be encrypted and the round key, and its output is XORed with the other half of the data. 
- This repeats a fixed number of times, and the final output is the encrypted data. The final round also performs a final swap.
- A major advantage of Feistel networks compared to other cipher designs such as substitution-permutation networks is that the entire operation is guaranteed to be invertible, even if the round function itself is not invertible
	- This means it can be made arbitrarily complicated. 
- Encryption and decryption are very similar, requiring only a reversal of the key schedule.
	 ![](Pasted%20image%2020260210222310.png)
### A single Feistel Round
- During each round, only **half of the block is encrypted**
- The round function takes the subkey $k$ and the right block $R_i$ and produces $f(R_i, k_i)$ 
- This output is $XOR$'d with the other half of the block to yield $L_i = f(R_i, k_i)$ 
	![](Pasted%20image%2020260210222514.png)
- The function $f$ should be a pseudorandom function ($PRF$).
	A $PRF$ is a family of functions: $$F_{k} : \{0, 1\}^n \to \{0, 1\}^m$$
	$k$ = key
	Efficient to compute; differs from a prng as this takes a key and an input, and will produce psuedorandom output  consistently for every input, whereas PRNG just takes a seed. PRNGs produce sequences, but $PRF$ focuses on the mapping. 

### Feistel Cipher Encryption
- Start with $L_{i} = L_i, R_{i} = R_i, k_{i}$
- Yield $L_i = R_i, R_i = L_i \oplus f(R_i, k_i)$ 
- Start with  $L_i = R_i, R_i = L_i \oplus f(R_i, k_i)$ 
- Yield $L_i = L_i \oplus f(R_i, k_i), R_i = R_i \oplus f(L_i \oplus f(R_i, k_i)k_{i+1})$ 
- Etc
- Final round does a final swap.
### Feistel Cipher Decryption
- Start with $L_i = R_i \oplus f(L_i \oplus f(R_i, k_i), k_{i+1}), R_{i} = L_i \oplus f(R_i, k_i)$  
	- After the function, we get $f(L_i \oplus f(R_i, k_i), k_{i+1})$, which when XOR'd with $L_i$, we simply get $R_i$, thus $R_i = R_i$ 
- This works because the XOR cancels out, and has nothing to do with whether the round function is invertible or not.

### About Feistel Networks
- 1 or 2 rounds are not sufficient to yield a cipher. 
	- A 1 round feistel on input $L_0, R_0$ is
		- $L_1 = R_0$
		- $R_1 = L_1 \oplus f(R_0, k_0)$ 
	- The ciphertext's left half equals the plaintext's right half, immediately distinguishing from the random permutation we are trying to achieve. 
	- A 2 round feistel is further distinguishable
- Luby & Rackoff proved that if the round function $f$ behaves like a secure $PRF$ then a 3-round feistel gives you a $PRP$. A 4-round Feistel gives you a strong $PRP$, meaning it stays pseudorandom even if an adversary can query both encryption and decryption. 
	- Meaning the attacker could interact with the cipher as a black box and ask it to process values of choosing
	- A cipher is a $PRP$ if an attacker who can only query encryption cannot tell whether they are talking to a real block cipher or a truly random permutation
	- A cipher is a strong $PRP$ if an attacker who can query both decryption and encryption cannot tell whether they are talking to a real block cipher and its inverse or a truly random permutation and its inverse. 
- Balanced Feistel networks
	- $∣L∣=∣R∣$ 
- Unbalanced feistel networks
	- $∣L∣≠∣R∣$
- Skipjack
	- Block cipher that uses a feistel like structure, but heavily exceeds the minimum 4 rounds, aiming to remain secure under very strong attack models
	- Shows how Feistel ideas are adapted in real ciphers. 
- OAEP
	- Padding/encoding scheme for RSA
	- Structurally similar to an unbalanced Feistel network
	- Mixes a short random seed and a long message block
	- Feistel ideas thus useful even outside of symmetric encryption. 

## Data Encryption Standard (DES)
- Symmetric key algorithm for the encryption of digital data
- Block cipher
- Key length of 56-bits ((64 bits with 8 parity bits, 1 bit in each byte utilised for error detection in key generation, distribution, and storage)too insecure for modern applications)
- Block size of 64-bits
- 16-round Feistel network
	- Guarantees invertibility
	- Allows you to reuse the same round function for encryption and decryption. 
- DES has been superseded by the advanced encryption standard

### Overall structure
![](Pasted%20image%2020260210235752.png)
- 16 stages of processing via Feistel network, termed rounds
- There is an initial and final permutation, termed $IP$ and $FP$, where these are their respective inverses.
	- $IP$ undoes the action of $FP$, and vice versa. Have no cryptographic significance. 
	- They merely specify a fixed bit rearrangement of the plaintext, so that  the first 32 bits naturally loaded into register $L$ and the next 32 bits loaded into register $R$.
	- This was for historical reasoning.
	- $FP / IP^{-1}$ is the inverse wiring.
		![](Pasted%20image%2020260210235719.png)
- $IP$ divided into two 32-bit halves (balanced) and processed accordingly. 
- As we know, the only difference in decryption is reversal in subkey scheduling. 


#### PC-1 and transformations
- Discard parity first and foremost
- We then rearrange the remaining key bits to spread key bits across different rounds
	- We achieve this by splitting the 56 bit key into two halves:
		$C$ and $D$.
	- Each round/transform, $C_i$ and $D_i$ are left-rotated (left shift with wrap around of shifted value - prepending the bit string).
	- Thus, we get a distinct round key per round, but the selection is deterministic
- As input to the Feistel function, 48 bits are selected, generally different bits omitted each round depending on the rotation history to prevent attacks that assume linear/simple key progression.
- 16 distinct 48 bit round keys $K_0...K_{15}$, every 1 bit change in the master key alters many round keys in many bit positions, thus strong **diffusion**.

#### The Feistel (F) Function
- The F-function operates on half a block and a subkey.
- Consists of 4 stages:
	1. **Expansion**
	2. **Key mixing**
	3. **Substitution**
	4. **Permutation**
##### Expansion
- DES needs 48 bits to be able to XOR with the 48 bit round key in key mixing, so it must create 16 extra bits
- Expansion cannot invent new information, so it must reuse existing bits
- And so roughly half of the 32 input bits appear twice in the 48 bit output, which adds diffusion. 
	![](Pasted%20image%2020260211001045.png)

#### Key mixing
- Expanded 48 bit value is $XOR$'d with the 48 bit round key $E(R_i) \oplus K_i$ where $E$ denotes the **expansion function.**
- $XOR$ is a linear function under $GF(2)$, we introduce non-linearity via the use of substitution boxes (S-boxes).
#### Substitution boxes
- S-boxes map 6-bit inputs to 4-bit outputs
	![](Pasted%20image%2020260211001755.png)
- There are 8 S-Boxes in total, each is different
- 48-bit input is split into 8 chunks of 6 bits, S-boxes applied to acquire 32-bit output
- Why does it not matter whether they're known??
##### S-box application
- Split the 6 bits into row bits and column bits
	- The first and last bits form the row bits
	- The column is formed from the middle 4 bits
		![](Pasted%20image%2020260211002025.png)
- Look up in S-box to determine 4 bit output
	![](Pasted%20image%2020260211002043.png)
	In the above case, would be $9$, $1001$. 
##### S-box design
- S-boxes need to be highly non-linear given that they introduce the non-linearity after XOR to prevent simple systems of linear equations being modelled to recover internal system state to predict output and break ciphers
- Key design principles include the following:
	1. No output bit should be too close to a linear combination of input bits
		Can become system of linear equations.
	2. 1 bit change in input should lead to at least 2 bits change in output
		Local avalanche.
	3. If only the middle 4 bits change, each output must occur exactly once
		Essentially, fix the row, vary the column through all 16 values, output must be a perfect permutation of 0-15
	4. If the first two bits are different but the last two are identical, the output must differ. 
	5. If two inputs differ by delta, their outputs should rarely differ by the same delta, differential cryptanalysis resistance.
	6. A collision is only possible for 3-adjacent S-boxes?

#### Permutation
![](Pasted%20image%2020260211004224.png)
- Given the Feistel equations $L_{i+1} = R_i$, $R_{i+1} = L_{i} \oplus F(R_i, K_i)$, the permutation $P$ is inside $F$.
- The 32 bit output from S-boxes are rearranged according to a fixed permutation, the $P-box$. This is designed so that after permutation, the bits from the output of each S-box in this round are spread across four different S-boxes in the next round. 
	Further aids confusion
- The next round's $F$ takes as input, $R_{i+1}$, so the permuted bits influence the $S-box$ inputs in that round.