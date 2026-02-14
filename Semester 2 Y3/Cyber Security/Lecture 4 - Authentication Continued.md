# Authentication
- Answers the question:
	"Is the **identified** subject who they claim to be"
- We distinguish **identification** to be the the selection of an identity and **authentication** to be a verification of that claim.
	- Authentication factors:
		- Something the user *is* (inherence)
		- Something the user *has* (possession)
		- Something the user *knows* (knowledge)
		- Somewhere the user is.
	- These categories differ not just in mechanism, but also in usability constraints, threat surface, revocability etc. 

## Inherence Factors (Biometrics)
- Biometrics are the statistical measurement of unique physical or behavioural (biological) characteristics to authenticate an individual's identity.
- Inherent to individuals
	- Their body (hands, eyes, face, voice)
	- Their behavuour(mouse movements, typing patterns, signature)
- "Suitable" biometrics for authentication have **desirable properties**:
	- **Universal** (almost everyone has it) and 
	- **Stable** over time
	- **Easily measurable**/computable and robust
	- **Uniquely** identifiable
	- **Secure**
### Examples of biometric authentication 
- **Fingerprints**
	- Extract points such as ridge endings, ridge orientation, relative distances between features, convert into mathematical template to be compared against during authentication. 
- **Facial recognition**
	- Extract distances between facial landmarks, analyse jawline contour, skin texture, 3D depth structure potentially. This is achieved via training a deep neural network to construct embeddings; numerical low-density feature vectors, so can compare vector similarity when authenticating. 
- **Iris/retinal scanning**
	- Unique texture patterns in coloured ring of eye, to be converted into an encoding, which can compute the distance between when authenticating. Needs special camera. 
	- Retinal scanning slightly different, measure blood vessel pattern at the back of eye, very difficult to fake, but is rather intrusive and expensive, not suitable for consumer products. 
- **Voice recognition**
	- Pitch, harmonic structures, speaking rhythm etc measured. Facilitates replay attacks however, need liveness detection, and in poor systems can deepfake. 
- **Behavioural biometrics**
	- Dynamics (mouse, typing habits)
		- Dynamics refer to the timing, rhythm, speed, and movement patterns of how someone performs an action over time.
		- Measureable features like speed, acceleration, timing gaps, pressure, etc
			- Typing bursts vs pauses, error correction habits etc.
	- Gait, posture
		- Gait = how someone walks, including step frequency, stride length, weight distribution, acceleration patterns.
	- Usage patterns
		- How someone interacts with a device over time. 
### Biometrics Authentication Process
![](Pasted%20image%2020260212213330.png)
1. **Capture**
	- Acquire raw biometric data from sensor e.g., fingerprint scaner, camera, iris scanner, microphone
	- Security
		  - Sensor quality affects accuracy + spoof resistance
		  - Liveness detection is critical
		  - Environmental noise e.g., lighting
2. **Feature extraction**
	- Extract discriminative features from raw biometric data e.g., facial embeddings via deep NN
	- Security:
		- Must tolerate intra-user variability via chosen representation and design threshold accordingly
		- Maximise inter-user separation
3. **Template creation**
	- Extracted features are converted to a template, typically a hashed representation or a compressed vector; feature summaries
	- Security:
		- Should templates be in plaintext or encrypted?
4. **Template storage**
	- Centralised database vs on-device vs hardware token
	- If centralised, templates must be encrypted at rest, access controlled
		- If hashed, do note that must allow approximate matching, hashing like passwords is not directly applicable. Might need to compute some other value.
5. **Comparison**
	- Captured features are extracted again and compared to stored template, and compute some sort of similarity/distance metric e.g., cosine similarity for unencrypted vector representations, hamming distance for iris codes, probabilistic modelling etc.
6. **Decision logic**
	- Design threshold:
		- $score \ge threshold \to accept$ 
		- $score \le threshold \to accept$ 
		- Threshold selected based on:
			- Security level required
			- Usability requirements
			- Regulatory constraints
			- Degree of feature extraction
	- Risk management decision
7. **Outcome**
	- Success, or fail, success may be fradulent, may lockour after repeated attempts etc. 

### Error tradeoffs
- **False Acceptance Rate(FAR)** 
	- Attacker wrongly accepted
- **False Rejection Rate (FRR)**
	- Legitimate user wrongly rejected.
- **Equal Error Rate** (ERR) is the point where $FAR = FRR$
	![](Pasted%20image%2020260212212845.png)
	Not necessarily optimal operational setting, used as comparative metric for biometric systems. 
	Depends on security requirements, for consumer FAR acceptable within low physical attack context, FRR minimised for usability, reversed for high-security environments e.g., border control. 

### Biometrics security considerations
- No risk of losing access on behalf of the user, inherent.
- Convenient where the system is well designed
- Availability of sensors
- Non-revocability
	- If fingerprint template leaks, cannot change fingerprint.
	- Fix: Cancellable biometrics, technique, distort biometric data via non-invertible transformations before storage, can cancel prior ones, and issue new one by changing transformation parameters. 
- Errors: FAR vs FRR
- Ethical concerns. 
- Not foolproof:
	- e.g., adversarial ML, small perturbations to model input to guide decision boundary e.g., slight audio modulation, slight pixel changes for facial recognition
	- Data poisoning e.g., backdoors in training data.

## Possession Factors
- Security credential based on something the user physically owns or has.
- The claimant controls a specific physical object that can be used for authentication purposes.
- Rely on physical control. 
- Often combined with knowledge factors or inherence factors to reduce the risk of unauthorised access. 
### Examples of Possession Authentication
- Keys (physical asset)
	- Literally just physical keys.
- **Hardware tokens**
	- Digitally implement the authentication factor of "something you have", proving the claimant controls a physical device that holds a **secret key**.
	- Strength of hardware tokens directly depend on cryptographic design and whether keys are extractable.
	- **One-time passwords** (can be device) (freshness, generate one every 10 seconds for example)
		- OTP token and server synchronised to same clock, generating new code every 30-60 seconds.
		- Prevent unauthorised access even if main password stolen, as OTP only valid for one session. 
		- Server side storage is dangerous, and manual entry introduces usability friction, can socially engineer too.
	- **Security keys**
		- Hardware based authenticator that stores private cryptographic key and performs cryptographic operations internally, solve password reuse, breaches, etc. 
		- Use asymmetric cryptography (see FIDO/WebAuthn)
- **Smart Cards**
	- Advanced hardware tokens, often with embedded microprocessors
	- May contain embedded private keys, tamper resistant hardware, pin activated protection. 
	- Access control, public transport, banking etc. 
	- Need card readers, deployment cost, loss or theft risk, hard to manage at scale.
- **Smartphone/wearables**
	- Modern smartphones can act as OTP generators, secure element containers.
		- Additional factors for smartphones include biometric unlock and trusted execution environment factors before even using the phone as a possession factor.
		- No extra hardware needed, and combine possession + biometrics + knowledge.
		- Single point of failure, and SIM swap risk.

#### Hardware tokens FIDO/WebAuthn
- During enrollment, the authenticator generates a public-private key pair
	- The private key is kept inside the device
	- The public key is sent to the server
- The server stores the public key along with some metadata e.g., credential ID, user handle etc. 
- During authentication:
	1. Server generates a challenge
		Pseudorandom, unpredictable value e.g., 32 bytes, unique per login attempt
	2. Browser sends challenge to authenticator
		Passes challenge + credential ID + website origin to device
	3. Hardware token signs the challenge
		Signs challenge with private key, and adds the website origin and metadata e.g., credential ID to the response
	4. Server verifies signature with the public key. 
- No shared secrets, no server-side credential secrets, slows phising via origin binding.

#### Hardware tokens: OTP vs Security Keys
- **OTP**
	- Hardware or digital (combine multiple auth factors usually)
	- Time/counter based OTP, temporary code that changes e.g., every 30 seconds
	- Prevent unauthorised access even if main password stolen, as OTP only valid for one session. 
	- Server side storage is dangerous, and manual entry introduces usability friction, can socially engineer too.
- **Security Keys**
	- Public-key cryptography, stored on key, and public key on server.
	- Challenge-response during authentication, which is automatic if paired with the authenticating device. 
	- Resistant to phishing, server attacks only yield public keys, secret is never compromised.

### Security Considerations
- Relies on strong keys and cryptography
-  Physical theft is difficult to scale, there for in principle very secure
- Automatic methods such as hardware tokens that are cryptography based are often part of MFA
	- Or their use even has implicit MFA e.g., knowledge/inherence for phone PIN/face recognition to access OTP app, 
- Physical loss/damage affects usability, difficulty of use also plays a role, sometimes good, sometimes not, depends on technical literacy of user
- Cost is an issue
- Theft is an issue. 


## Knowledge Factors
- Implement the "Something you know" authentication principle/method
- Examples include passwords, passphrases, PINs, security questions
- The core assumption is that only the legitimate user knows the secret.
- Knowledge factors are based on **shared secrets**:
	- The user knows a secret
	- The server/authority stores a representation of that secret
	- Authentication succeeds if the user proves knowledge of it, creating a symmetric trust model. 
- The theoretical strength is tied to **entropy**
	- Entropy measures the unpredictability of a secret in tandem with the size of the search space. 
		- Security questions must be designed such that answers yield high entropy.
	- In theory, generated passwords over a large search space e.g., $25^{20}$ 
- But in practice, human-generated secrets have very low entropy, usually predictable regardless of search space size. 
	- Password reuse, predictable passwords, modify incrementally, use dictionary words.
	- Can be **guessed, cracked, stolen, misused**
	- Furthermore, not be well managed server side. 
- The dispersion of knowledge factors is due to the fact it strikes a balance between availability, portability, convenience, and cost, despite its security flaws. 
	- Universal, low cost, revocable, portable
	- Vs scalable attack surface, phishable, server-side security must be strong.

### Attack Vectors Against Knowledge Factors
- **Offline cracking if database is breached**
	- Only defenses are really salting and slow hashing, but slow passwords fall quickly, even if rainbow table attacks are prevented.
- **Phishing**
	- User voluntarily enters secret into fake site, secret is then transferable.
	- Catastrophic if password reuse in play
- **Credential stuffing**
	- Password reuse, breach on one site, can be used automatically on others, secret reuse causes this. 
- **Social engineering**
	- Particularly effective for security questions, typically low entropy and can be easily guessed with small amounts of information. 

### Password managers
- Generate high-entropy passwords and avoid reuse, storing securely in an encrypted vault
- Aim to reduce credential stuffing and improve entropy dramatically to make offline cracking and other attack vectors less feasible. 
- The large risks are:
	- Master password compromise
	- Malware on host
	- Impact of vault breach is catastrophic for user
	- Third-party trust needed - attackers may even masquerade as legitimate password manager then exit scam. 
	- Essentially trade some forms of attacks for others
- In the balance, probably a positive