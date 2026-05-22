### DES and cryptanalysis
![](Pasted%20image%2020260211005300.png)

### DES key schedule
- A **key schedule** is a component of a block cipher that takes one master key and turns it into a series of subkeys; one per round
- The DES key schedule simply returns various permutations of $k$ as subkeys


### Permuted Choice 1
- Permuted choice 1 is responsible for selecting 56 of 64 bits of the master key $k$; 8 parity bits, one per byte. 
	![](Pasted%20image%2020260211010053.png)
	Key storage, generation, distribution

### Left rotations and splitting 
- Take the 56 bit value after $PC-1$  has been applied and split into 2 halves $C_0$ and $D_0$ 
- Left rotation, leftmost numbers wrap around to RHS instead of being discarded denoted via $<<<n$ 
- 16 rounds of rotations(1 per subkey)
	- In DES, $C_i$ and $D_i$ are rotated left $<<<1$ for rounds $\{1,2, 9, 16\}$  and $<<< 2$ for all others
	- The total rotations is $4 \cdot 1, 12 \cdot 2 = 28$ schedule h**as clean periodicity**; the key halves are back in their original positions, meaning there is no long term evolution of the key and its management is simplified. 

### Permuted Choice 2
- Selects **48** of the **56** bits to be used as a round key
	- Generally omit different bits depending on rotation history to prevent attacks that assume simple/linear key progression and schedule. 

### Properties of the key schedule
- Entirely permutation based; no XOR, addition, mixing, just reorders bits and selects some
	- Due to this, one master key bit only inherently can affect one output position and therefore across the rounds there is generally weak diffusion. 
	- Relationship between keys may be predictable, especially if permuted choice 2 remains fixed. 
- $C_0 = C_{16}, D_0 = D_{16}$ 


### Breaking DES
- Means we can recover the secret key by trying all possiblilites in a feasible amount of time
- Attack model = **Known-Plaintext attack**
	- Attacker has the algorithm
	- One or more plaintext-cipher pairs
		- Realistic e.g., file headers, protocol messages have fixed structure, easy to obtain if you have some additional info and packet sniffer
	- No encryption oracle - cannot ask real system for encryption under key trying to uncover to determine some relationship between it and the plaintext.
- A **brute-force attack** under this model only requires the pair $(x_0, y_0)$
	Test keys:
	$DES^{-1}k_i(y_0) = x_0, i=0...2^{56-1}$ 
		Breaking DES via a known-plaintext bruteforce attack is equivalent to the size of the key searchspace
		Recall that a block cipher, for a random secret key aims to emulate a pseudorandom permutation that is a bijection to an adversary without the key; thus, the key space cannot be assumed to be halved etc, so it is exhaustively searched
		Can be parallelised; on modern GPUs, can be broken


### Key Collision
- Two different keys $k \neq k^*$ both satisfy $E_k(x_0) = y_0$ for the same known-plaintext ciphertext pair. 
- So during brute force, attacker tests key, works for known pair, but may be incorrect key. 
- An ideal block cipher = family of random permutations such that under $\{0,1\}^n$ a cipher of equivalent size is yielded that is indistinguihsable from a random permutation of the input
- The probability that a worng key matches a known pair:
	- Block size $n$ bits
	- Ciphertext size $2^n$ 
	- Number of total keys $2^l$ 
	- $\frac{2^l}{2^n}$ = expected number of false keys, 1/256 for DES, so brute force often stops  when the first key match found. 
		- Under the ideal PRP assumption where the block cipher is a biject over $k, x_0$, equally likely to be any $y_0$ and thus $Pr[E_k(x_0) = y_0] = 1 / 2^n$ 
- Can just verify with multiple pairs

![](Pasted%20image%2020260521215350.png)



## Double encryption
- Encrypt with DES twice, each with 2 independent 56-bit master keys:
	$y = Enc_{k_{1}}(Enc_{k_{0}}(x))$ 
	![](Pasted%20image%2020260211024118.png)
- The naive brute force approach suggests there are 2^56 choices for $k_0$ and $2^{56}$ choices for $k_1$, therefore the total pairs to enumerate $=2^{56} \cdot 2^{56}$ 
- However because the cipher has a **meeting point** (the intermediate ciphertext after the initial encryption) this can be exploited to avoid exhaustively searching the cartesian product search space. 

### Meet in the middle attack DES
1. Calculate all encryptions of $x_1$ under $K_{L, i}$ and store in intermediate values $Z_{L, i}$ 
2. Calculate all decryptions of $y_1$ for all $K_{R, j}$ to compute $Z_{R, j}$
3. Compare $Z_{r, j}$ matching existing $K_{L, i}$ 
	![](Pasted%20image%2020260211024937.png)
	You compute independently, and then compare, going forwards and backwards to meet in the middle via precomputed tables
	This is much more efficient than exploring the search space via cartesian product yielding $2^{57}$ search space for DES, or more generally for similar constructions $2^{k+1}$ 
	It trades off **computation speed for storage** - in the order of **petabytes for DES** and the model assumes some kind of $O(1)$ lookup for $Z_{L, i}$  and $Z_{R, j}$ 


### Alternative DES constructions
#### 3DEs
- Use three different keys.
	- Either $E_{k_3}(E_{k_2}(E_{k_1}(x)))$ or $E_{k_{2}}(D_{k_{2}}(E_{k_{1}}(x)))$
	![](Pasted%20image%2020260211032751.png)
	- We tend to use $EDE$; the reason being is that if all keys end up to be the same then it collapses the $E_k(x)$ giving backward compatibility when using this keying option
	- $D_{k_{2}}$ is not a notation mistake, it is an intentional inverse permutation for given key $k_2$ applied to $E_{k_1}$ 
		- Recall that this would just reverse the order of the key schedule and start with the final halves of the first encryption operation, so in essence providing this isn't the same key it behaves functionally the same as DES.
	- Each triple encryption encrypts one block of 64 bit data

### MITM
- 2DEs, single clean middle point to collapse search space into
- 3DES, 2 internal boundaries, three independent keys
- Approaching 3DES in the similar manner fails. 
	- You cannot enumerate $k_2$ cleanly. YOu could try and do $k_1$ and $k_3$ and compute all respective encryptions and decryptions but this leaves you with a middle component which depends on a further unknown key which you'd have to test against both which is actually worse than brute force.


### DES-X
- Leverages **key whitening** - technique that increases the security of a block cipher, consisting of steps to combine data with portions of the **key**
- Most common form of key whitening uses two extra 64 bit keys, resulting in a **XOR ENCRYPT XOR** pattern. 
	- XOR with plaintext, then encrypt over that, then XOR again
		![](Pasted%20image%2020260521222214.png)
- 
- Increase effective size of the key without major changes in the algorithm, theoretically providing a search space of $2^{k+n}$ where $n$ is the size of non-encrypt keys. 
	- Because whitening uses XOR, it is much easier to exploit this structure
	- If known plaintext conditions, the structure can be exploited
	- Very much similar to 2DES MITM, can split into two and match intermediate values
		- Differential cryptanalysis gives the following security as 
		- $2^{k+n-m}$ where $m$ is the number of $2^m$ plaintexr-ciphertext mappings the attacker has
		- Compute all $2^{56}$ keys over some plaintext $x_0$ and store the results, then for $k_1$ and $k_2$ work forwards, and backwards from either side, until a match is reached. 


### Cryptanalysis
### Cryptanalytical attacks
- **Attack models** describe the capabilities and knowledge an attacker is assumed to have when trying to break a cryptographic system
- Used to evaluate how strong a cryptographic primitive or protocol is in practice
	1. **Brute Force**
		- Knows the algorithm and tries keys until one works, usually assumes at leat one plaintext/ciphertext block to compare against
	2. **Ciphertext only**
	3. **Known-plaintext**
		- Attacker has a few $2^m$ plaintext ciphertext mappings - more than in brute force
	4. **Chosen plaintext**
		- Attacker is able to query an encryption oracle with their chosen plaintext under $k$.
	5. **Chosen ciphertext**
		- Attacker can query a decryption oracle with their chosen ciphertext under $k$
	6. **Related key attack**
		- Instead of attacking single unknown key, an attacker is able to study encryptions made with different keys that are mathematically related
		- Theoretical concern

### Break
- In modern cryptography, a cipher is declared to be **broken** if there is any attack that is more efficient than brute force. 
- E.g., differential cryptanalysis requires $2^{47}$ operations on DES and not $2^{56}$ and thus it is **broken**
- Often academic breaks rather than practical security concern
	- E.g., requiring unrealistic amounts of data, many chosen plaintexts, stronger attack model

### Analytical
- Exploits structural or mathematical weakness directly
	- MITM attacks, structure of double encryption to effectively halve the key space
	- LSFR tap recovery via conversion into system of linear equations after acquiring some key bits under known plaintext attack model


### Statistical
- Capture statistical, non-uniform probabilities (not perfectly random)
- **Differential cryptanalysis**
- **Linear cryptanalysis**

### Differential cryptanalysis
- Most important method for breaking block ciphers
- Differential cryptanalysis constitutes a **chosen-plaintext attack** aiming to find predictable output changes caused by known input changes
	- Choose two plaintexts, observe their differences, encrypt both, observer differences in ciphertext, can make some guesses, especially when know algorithm. 
	- $S-boxes$ take a 6 bit input and produce a 4 bit output
		- For any input difference $Δx$ the output difference $Δy$ should occur with probability $\frac{1}{16}$ (4 bits)
		- Some input change resulting in some output change is called a **differential** and has some probability of occurring.

### Differential cryptanalysis in DES
- Choose some plaintexts with input delta $\Delta x$ and encrypt them
- Look for $\Delta y$ in output and determine the probability (though, of course, this is permuted via P-box and goes through multiple rounds where the input halves change and subsequently diffuse, so isn't trivial to follow)
- If the differential is not a uniform $1/16$ then can begin to guess subkey bits

### Resisting Differential Cryptanalysis in DES
- S-boxes should be designed such that the probability of any differential is as low as possible. 
- More rounds, makes differentials less likely.
- Good permutation. 


