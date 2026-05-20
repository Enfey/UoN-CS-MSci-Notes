## Authentication
- ***Process of verifying an entity is who they claim to be***
	- So access control decisions + comms decisions, can be made
- To grant access to a system, must ensure the user is:
	- Identified
	- Authenticated
	- Authentication may expire, recheck
- Authentication, relies, **credentials**:
	- *Something you know* - password, pin
	- *Something you have* - tokens, keycard, MFA
	- *Something you are* - biometrics
- Often combine

### Usernames vs Passwords
- Username = identification
- Password = authentication
- Auth should expire
	- Remember my credentials options - turns into something you have (cookie)
	- Authentication often repeated during session, prevent **session hijacking**
		- TOCTTOU - class of bugs, caused by race condition, involving verifying a resource(e.g., checking auth to be able to manipulate file), but a time gap occurs before the resource is acted upon. Attacker can influence that gap and hijack e.g., swap a legitimate file with a malicious one. 
		- Attackers, only need, win once, make small windows, exploitable, especially via automation.
			- E.g., check = session token at start of request, attacker can change session state before request finishes executing
			- For example, 1 thread observes the resource (verifies user is allowed to perform action), but another thread runs and overwrites that shared memory with another users info, then then the initial thread performs the action with the users identity, can easily be carried out by sending multiple requests to a server.


## Problems with passwords
- Passwords fail due to human behaviour e.g., forget passwords, reuse, guessed, phished, keylogged. 
	- Weak passwords make issues worse.

### Weak passwords
- Short, dictionary based, predictable, may contain personal info, often recurring (data breaches show many users choose same passwords)
- Users don't understand security - system is sound, the individual isn't

### Storing passwords
- **Plaintext**
	- If breached, catastrophic
	- Admins can read
	- Single point of failure in all cases.
- **Encrypted**
	- Still reversible, encryption key must be stored
	- If comrpomised, all passwords = exposed(unless using diff algos, can harden, but overhead of safe key storage vs hashing just not worth it)
- **Hashed**
	- One way cryptographic functions
	- Take input of arbitrary size, return **hash/digest** of fixed size
	- Infeasible to reverse, strong avalanche characteristics
			$h(M):\{0, 1\}^n \to \{0, 1\}^{128}$
	- The critical properties are:
		- **Preimage resistance** - given a hash must be infeasible to invert it, to original input. 
			- This is the form that attacks on stolen hashed databases take
		- **Collision resistance** - it should be computationally infeasible to find any two inputs that produce the same hash output
		- **Output must be indistinguishable from random noise**
	- We store hashes of passwords; when authenticating, hash the provided password and compare to the stored one.
	- Should DB leak, only hashes compromised.
	- If an attacker gets hashes, can perform **offline attacks**
		- No rate limiting, just perform guesses
		- On OSs steps taken to stop people reading hashes for offline attacks;
			- Linux /etc/shadow has salts + hashes; /etc/paswd has username UID, GID, shell, home dir
			- Only root can read /etc/shadow
			- Assumes admins trusted.


### Cracking passwords
- Process of recovering plaintext passwords either:
	- **Online**
		- Live authentication interface
	- **Offline**
		- Already have hashes locally, performing preimage attack
- Cracking not illegal; is illegal without authorisation. 
- **OFFLINE**
	- **Threat model** - attacker has hashes, worst case scenario, defence based on hash design + salting
	- No rate limiting, detection; can parallelise search
	- **Techniques**
		- Offline password cracking, try possible passwords, hash, see if have hash collision using same algorithm and salt. 
			- Use password list to try random passwords
		- **Brute Force**
			- Search space = $\{char\_count\}^{length}$
			- Infeasible for long passwords
		- **Dictionary attacks**
			- Dominant method, do not choose at random, maintain dictionary, common words + passwords, sample from that, perturb, combine etc.
	- Password strength determined by **search space** and how well **elements in that search space are structured.** e.g., ban predictable patterns, permitting symbol complexity.

### Password salting
- Can improve password security by prepending a random value to the password.
- This salt is unique for each user; does not go into hash function, and is stored in plaintext alongside the hash. 
- This prevents hash collisions between users decreasing the scale of the attack, and prevents pre-computed rainbow table attacks because such table don't include the salt
	- Forces the attacker to compute guesses individually for each user, makes practically impossible to crack on large scale
![](Pasted%20image%2020260205201300.png)


## Hashing speed
- When password cracking, the most important factor is **hashing speed**
- Measured in (H/s)
	- Faster hashing  = shorter cracking time
- New algos, take longer, e.g., bcrypt, designed to take a while, slow down offline attacks by introducing non-GPU friendly hashing, salting, and large numbers of iterations. 

### Pretexting
- Social engineering attack; attacker invents story, the **pretext** and fake identity, convince target that request is legit. 
- Goal = extract sensitive info e.g, passwords, OTP, personal details, account numbers etc
- Exploits trust so:
	- Verify identities, avoid sharing sensitive details over unsecure channels, use official comms channelsm etc.