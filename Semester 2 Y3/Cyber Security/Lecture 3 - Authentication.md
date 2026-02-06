# Authentication
- **Authentication** is the process of verifying that an entity is **who they claim to be**, so that access control decisions and communication decisions can be made.
- Generally, to grant access a system must ensure the user is:
	- Identified
	- Authenticated
	- Authentication may need to expire and be rechecked.
- Authentication relies on **credentials,** typically:
	- Something you know(passwords, pins), something you have(tokens, phone MFA), or something you are (e.g., biometrics)
- Modern systems often combine these.

## Usernames vs Passwords
- Username = identification
- Password  = authentication
- Authentication should expire
	- Remember my credentials turns authentication into *something you have(cookie or a token)*
	- Authentication often repeated during a session to prevent session hijacking
		- Time-of-check to time-of-use are a class of bugs caused by a race condition involving checking the state of part of a system (e.g., credentials) and then later using that result, and the property changes in between. If an attacker can influence/interpose the gap between authentication and use, they can break security guarantees.
		- Attackers only need to win once, automation makes small windows exploitable; say a user logs in, and the server issues a session token, if that session token is stolen, the attacker can reuse it; the system may not re-check identity, causing the system to apply outdated permissions. 

## Problems with passwords
- Most passwords fail due to human behaviour, not cryptography
- E.g.,
	- Forget passwords
	- Reuse passwords
	- Passwords are guessed, phished, or keylogged
	- Password databases get stolen
- Weak passwords massively amplify these issues

### Weak passwords
- Tend to be short, based on dictionary words and are predictable, may also contain personal information
- There are recurring common sequences
- Real-world data breaches show that millions of users choose the same passwords, making attacks extremely efficient
- Users dont understand security, the weakness is not the system, but the individual. 
### Storing passwords
- **Plaintext storage**
	- If breached, catastrophic
	- Admins can read passwords
	- In all cases, single point of failure, very worrying!
- **Encrypted passwords**
	- Still reversible, the encryption key must be stored, and if this is compromised, then all passwords are exposed (unless using multiple different algorithms etc, can make more complex to compromise, but the key management scheme overhead is costly compared to the hashing alternative).
	- Admins can still read them
- **Hash Functions and password storage**
	- One way cryptographic functions
		- Take input of arbitrary size, and return a **hash** (digest) of fixed size
		- Infeasible to reverse, with strong avalanche characteristics
			- $h(M):\{0, 1\}^n \to \{0, 1\}^{128}$
		- The critical properties are:
			- **Preimage resistance** (cannot reverse)
			- **Collision resistance** (solved by mathematically guaranteeing strong avalanche characteristics).
			- Output must be **indistinguishable from random noise.**
		- We can then store these hashes; when authenticating, hash the provided password, and compare the hashes
		- Should the DB leak, only hashes are compromised.
		- If an attacker gets **password hashes**, they can perform offline attacks:
			- No rate limiting/logging vs online guessing. 
			- On OSs steps have been taken to stop people reading hashes for offline attacks, linux stores in /etc/shadow, and /etc/passwd stores username, UID, GID, shell, home dir, with no hashes, shadow stores the salts and hashes, and only root can read /etc/shadow. 
			- This assumes admins are trusted.

### Cracking passwords
- Process of **recovering plaintext passwords** from either
	- **Online:** 
		- Live authentication interface
	- **Offline:**
		- Stored password verification material (already have the hash locally)
- Cracking is not inherently illegal, security professionals crack for auditing, forensics etc, becomes illegal when done without authorisation. 
- **OFFLINE**:
	- **Threat model**: the attacker has obtained password hashes, this is the worst-case scenario, the defense focus here is on hashing design and salting. 
	- Attacker has no rate limiting, no detection, potential for massive parallelism e.g., GPUs and distributed computing
	- **Techniques**:
		- Offline password cracking is quite simply a case of trying possible passwords, and seeing if we have a hash collision using the same algorithm and salt, we use a password list to try random passwords.
		- May be a **brute force** approach
			- Search space = $\{char\_count\}^{length}$
			- GPUs are fast, but not fast enough for long passwords.
			- Try every possible combination, becomes infeasible for long passwords. 
		- **Dictionary attacks**
			- Dominant method, do not choose passwords uniformly at random, but instead maintain a dictionary of common words and passwords.
			- Perturb dictionary entries, combine words etc
			- Exploit structure of passwords.
	- Thus, password strength is determined by effective search space, and how well elements in that search space are structured e.g., banning predictable patterns, permitting symbol complexity. 
		![](Pasted%20image%2020260205200802.png)
		10/20k word dictionary

### Rainbow tables
### Password salting
- Can improve password security by prepending a random value to the password before hashing. 
- The salt is unique for each user and stored unencrypted with the hash.
	![](Pasted%20image%2020260205201300.png)
- Prevents hash collisions between users, decreasing scale of the attack, and prevents pre-computed rainbow table attacks because such tables dont include the unique salt, forcing the attacker to compute each guess individually for each user, makes less computationally feasible to crack. 
### Hashing speed
- When password cracking, the most important factor is **hashing speed**.
- Measured in hashes per second (H/s) - how many password hashes per sec
	- The faster the hashing, the shorter the cracking time
- Newer hashing algorithms take longer (e.g., bcrypt, PBKDF2)
	- More complex, and some have been specifically designed to take a while
	- With a slow hash e.g., 5000H/s, cracking time would be roughly equal to 1520 yrs for 240 trillion search space. 
- Bcrypt, designed to make offline password-cracking extremely difficult
	- Automatic salting
	- Cost factor
	- Blowfish-based hashing, which is not GPU-friendly, making bcrypt stronger against GPU attacks
- PBKDF2 transforms a password into secure, hard to crack hash with
	- A salt
	- A large number of iterations
	- A cryptographic hash function e.g., SHA-512

### Pretexting - If cracking fails
- Social engineering attack where an attack invents a believable story, the pretext, and a fake role/identity, to convince a target that a request is legitimate
- Goal is to extract sensitive information e.g., passwords or one-time codes, personal details, account numbers, internal procedures etc.
- Pretexting exploits trust so:
	- Verify identities before sharing info
	- Avoid sharing sensitive details over unsecure channels
	- Use official comms channels
	- Report suspicious requests