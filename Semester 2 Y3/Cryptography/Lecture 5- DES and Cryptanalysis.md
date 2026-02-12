![](Pasted%20image%2020260211005300.png)

# DES Key schedule
- A key schedule is a component of a block cipher that takes one master key and turns it into a list of round/sub keys, one per round
- The DES key schedule simply returns various permutations of $k$ as sub-keys, $k_0, ...,k_{15}$
## Permuted Choice 1 (PC-1)
- Permuted Choice 1 is responsible for selecting 56 of the 64 bits
- DES keys typically written as 64 bits, but 8 are parity bits, the last bit in each byte is used for error detection in key storage, distribution, and generation
	![](Pasted%20image%2020260211010053.png)

## Left Rotations and Splitting
- Take the 56 bit value after PC-1 and split into 2 halves $C_0$ and $D_0$, each 28 bits wide.
- Left rotations represent a left shift where the leftmost numbers wrap around to the right hand side instead of simply being discarded.
	![](Pasted%20image%2020260211010958.png)
- 16 rounds of rotations(1 per subkey):
	- In DES, $C$ and $D$ are rotated left $<<<1$ for rounds $\{1, 2, 9, 16\}$ and $<<<2$ otherwise
	- The total rotation is $4 \cdot 1 + 12 \cdot 2 = 28$, schedule has clean periodicity; after all 16 rounds of rotations, the key halves are back in their original positions. 
		- No long term evolution of the key.

## Permuted Choice 2
- Permuted choice 2 selects 48 bits of the 56 bits to be used as a round key.
	- Generally can omit different bits each round depending on the rotation history to prevent attacks that assume linear/simple key progression.

## Properties of the key schedule
- Entirely permutation based, with no XOR, addition, mixing, only reorders bits and rotates them. 
	- Due to this, one master key bit only affects one bit position as there is no diffusion mechanism and therefore weak avalanche characteristics. 
		- If you flip one master-key bit, exactly one bit flips in each round key where that bit appears.
	- Relationship between keys may be predictable, especially if permuted choice 2 remains fixed. 
- $C_0 = C_{16}$ and $D_0 = D_{16}$ 

# Breaking DES
- Breaking here means that we can recover the secret key by trying all possibilities in a feasible amount of time
- This attack model is assumed as being a **Known-plaintext attack**
	- The attacker has the algorithm (DES is a standard)
	- One or more plaintext-cipher pairs
		- Realistic, file headers, protocol messages have fixed structure e.g., timestamps, padding etc are known. 
	- No encryption oracle during the attack
		- That is, the attacker cannot query the real system and ask for encryption under the unknown secret key $k$.
		- Run the public algorithm locally with guessed keys instead. 
- A brute-force attack under this attack model requires only the pair $(x_0, y_0)$ of plaintext and ciphertext. 
- Test keys: $DES^{-1}k_i(y_0) = x_0$, $i = 0, 1, ..., 2^{56-1}$ 
	- Breaking DES via a known-plaintext bruteforce attack is proportional to the size of the key space.
	- Recall that a block cipher, for a random secret key, aims to emulate a random permutation to an adversary without the key. Therefore, exhaustively search the key space, which is feasible ($7 \times 10^{16}$ possbilities, can easily parallelise this.) 

## Key Collisions
- Two different keys $k \neq k^*$ both satisfy $E_k(x_0) = y_0$ for the same known plaintext-ciphertext pair $(x_0, y_0)$.
- So during brute force, attacker tests key, works for known pair, but may be incorrect key. 
- An ideal block cipher is a family of random permutations, such that for a fixed plaintext, and a given key, the output $E_k(x_0)$ looks like a random n-bit string, and that a $PRP$ is bijective. But this is not often achievable, and settle for an approximation.
- The probability that a wrong key matches a known pair:
	- Block size = $n$-bits
	- Ciphertext space size = $2^n$ 
	- Number of total keys = $2^l$ 
	- $\frac{2^l}{2^n}$ = expected number of false keys, 1/256 for DES, so brute force often stops when first key match found.
		- Extra detail:
			- Under the ideal $PRP$ assumption, where the block cipher is bijective over $k, x_0$, equally likely to be any of the 2^n ciphertexts, thus $Pr[E_k(x_0) = y_0] = \frac{1}{2^n}$ 
	- Can just verify a candidate key using more than one known pair. 
## Double encryption
- Encrypt with DES twice, with 2 independent 56-bit master keys:
	- $y = E_{k_{R}}(E_{k_{L}}(x))$ 
		![](Pasted%20image%2020260211024118.png)
- The naive brute force approach suggests there are $2^{56}$ choices for $k_l$, and $2^{56}$ choices for $k_R$, therefore total pairs is $2^{56} \cdot 2^{56} = 2^{112}$, try every key-pair. 
- However, because the cipher has a **meeting point** (the intermediate ciphertext after $E_{k_{L}}(x)$, this can be exploited to avoid exhaustively searching the cartesian product search space. 

## Meet in the middle
- Meet in the middle attack splits the search space in two:
	1. Calculate encryptions of $x_1$ for all $k_{L,i}$ and store intermediate values $Z_{L, i}$ 
	2. Calculate all decryptions of $y_1$ for all $k_{R, j}$ to compute $Z_{R, j}$ 
	3. Find any value of $Z_{R, j}$ matching existing $Z_{L, i}$ 
	![](Pasted%20image%2020260211024937.png)
- This requires $2^{k+1}$ attempts rather than $2^{k \cdot 2}$, performing key search twice.
- This is much better than brute force, but does not make it straightforward.
- Trades off computation speed for storage - in the order of petabytes for DES, and this model further assumes some kind of $O(1)$ lookup $Z_{L, i}$.
 
# Alternative DES constructions
## 3DES
- Use three different keys
	- Either $E_{k_{3}}(E_{k_{2}}(E_{k_{1}}(x)))$ or $E_{k_{2}}(D_{k_{2}}(E_{k_{1}}(x)))$![](Pasted%20image%2020260211032751.png)
	- The reason we tend to use EDE is that if all keys are the same, then 3DES collapses to single DES, $E_k(x)$, providing backward compatibility when using this keying option.
	- It should be noted that $D_{k_{2}}$ is not a notation mistake, is it an intentional inverse permutation for given key $k_2$ applied to $E_{k_{1}}(x)$.
	- Decryption is just that sequence reversed. Each triple encryption encrypts one block of 64 bits of data. 
### MITM
- For 2DES, there is a single clean middle value to collapse the search space unto.
- In 3DES, there are two internal boundaries and three independent keys. 
- Approaching 3DES in the same manner fails.
- There are two internal values
	- May try to enumerate $k_1$ and $k_3$, and compute all respective encryptions and decryptions, but this leaves you with the middle component which depends on a further unknown key, this would require testing $(k_1, k_3)$, and then searching over all $k_2$, becoming $2^{3k}$ which is worse than brute force. 

## DES-X
- DES construction that leverages **key whitening** - a technique intended to increase the cryptographic security of an iterated block cipher, consisting of steps that combine thee data with portions of the **key**.
- The most common form of key whitening, and the one used by DES-x, simply uses two extra 64-bit keys, resulting in a $xor-encrypt-xor$ pattern.
- Increase effective size of the key, without major changes in the algorithm, theoretically providing a search space of $2^{k+n}$ where $n$ is the size of non-DES keys.
	- Because whitening uses XOR, which is linear under $GF(2)$, much easier to exploit this structure. If the attacker knows plaintext-ciphertext pairs, structure can easily be exploited to break parts of the cipher. 
	- Meet in the middle further works, can split into two and match intermediate values similar to 2DES.
- In practice security is $2k^{k+n-m}$where an attacker has $2^m$ known plaintexts.

# Cryptanalysis
## Cryptanalytical attacks
- **Attack models** describe the capabilities and knowledge an attacker is assumed to have when trying to break a cryptographic system.
- Used to evaluate how strong an algorithm is.
- From weakest attack strength to strongest:
	- **Brute force**
		- Knows algorithm tries keys until one works, usually assumes at least one plaintext/ciphertext block to test candidate keys
	- **Ciphertext only**
	- **Known-plaintext**
		- Usually more than in brute force. 
	- **Chosen plaintext**
		- Attacker can choose plaintexts and obtain ciphertexts **under the secret key** ("encryption oracle").
	- **Chosen ciphertext**
		- Attacker can submit ciphertexts and get decryptions ("decryption oracle")
		- Typically some restrictions?
	-  **Related key-attack**
		- Instead of attacking single unknown key, attacker studies encryptions made with different keys that are mathematically related
		- Attacker observes ciphertexts under multiple keys
		- Knows how the keys are related mathematically.
		- Theoretical concern. 

## Break
- What is a break?
- In modern cryptography, a cipher is declared broken by essentially any attack that is more efficient than brute force
	- If brute force is $2^k$ then we seek for no better method. 
	- For example, differential cryptanalysis requires $2^{47}$ operations on DES rather than $2^{56}$ and thus it is is **broken**.
- These are often academic breaks, rather than a practical security concern:
	- For example, requiring unrealistic amounts of data e.g., many chosen plaintexts, or depend on a strong attack model such as related key or chosen ciphertext.
	- Related key attack on AES of $2^{99.5}$ compared to brute force $2^{128}$ 
- $2^{n-1}$ = half the time required as $2^n$

### Analytical attacks
- Exploits structural or mathematical weakness directly
	- Meet-in-the-middle attacks composition structure of double encryption to effectively halve the key space.
	- LSFR tap recovery via the exploitation of its linear recurrence structure via gaussian elimination is another.
### Statistical attacks
- Capture statistical, non-uniform probabilities (the cipher behaves almost random, but not perfectly).
	- Differential cryptanalysis
	- Linear cryptanalysis

## Differential cryptanalysis
- Perhaps now the most important modern method for breaking block ciphers. 
- **Differential cryptanalysis** constitutes a **chosen-plaintext** attack aiming to find predictable output changes caused by known input changes. 
- 
