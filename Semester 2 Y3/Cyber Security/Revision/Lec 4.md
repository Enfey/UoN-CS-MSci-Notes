## Auth
- Answers question: **is identified subject, who they claim to be?**
- Distinguish identification to be the selection of an identity, and authentication to be the verification of that identity
	- Auth factors:
		- Something the user knows (**knowledge**)
		- Something the user has (**possession**)
		- Something the user is (**inherence**)
	- These categories differ in mechanism, usability, threat surface, revocability etc.

## Inherence Factors
- **Biometrics** = statistical measurement of unique physical or behavioural characteristics to authenticate an identity.
- Inherent 
	- **Biology** - hands, eyes, face, voice
	- **Behaviour** - mouse movements, typing patterns, signature.
- "Suitable" biometrics for authntication have **desirable properties**:
	- **Universal** - everyone has
	- **Stable over time**
	- **Easy to measure**
	- **Unique**
	- **Can secure**

###  Examples of biometric auth
- **Fingerprints**
	- Extract points such as **ridge endings, ridge orientation, relative distance between features**
	- Convert to **mathematical template**, store, and compare
- **Facial recognition**
	- Extract distasnces between **features, analye jawline, skin texture, 3D depth structure**
	- Train NN, construct embeddings, **compare vector similarity when auth.**
- **Iris/retinal scanning**
	- Iris - **Unique texture patterns**, coloured ring of eye, encode, and auth. Need special camera
	- Retinal - measure **blood vessel pattern** at back of eye, very difficult to fake, intrusive + expensive, not suitable, consumer product. 
- **Voice recognition**
	- Pitch, speaking rhythm etc, facilitates replay attacks however, need **liveness detection**. can deepfake  in poor systems
- **Behavioural biometrics**'
	- **Dynamics**(mouse, typing habits)
		- Refers to timing, rhythm, speed, and movement patterns of how someone performs action over time. 
			- Acceleration, speed, timing gaps, pressure e.g., typing bursts vs pauses
	- **Gait/posture**
		- Gait = how someone walks iincluding step frequency, stride length, weight distribution. 

### Biometrics authentication process
1. **Capture**
	- Acquire raw biometric data from sensor e.g., fingerprint scanner, camera, mic
	- Security
		- Sensor quality affects accuracy + spoof resistance; need liveness detection, and immunity to environmental noise e.g., lighting
2. Feature extraction
	- Extract features from raw data e.g., facial embedding vectors via deep NN
	- Security:
		- Tolerate intra-user variability via chosen threshold
		- Maximise inter-user separation?
3. **Template creation**
	- Extracted features, converted, template, typically hashed representation, feature summaries
4. **Template storage**
	- Centralised DB vs on-device vs ..., access control + encryption if centralised
	- If hashing, must allow approximate matching, hashing not work exactly the same. May compute some other val too. 
5. **Comparison**
	- Capture raw data, extract features, compare against stored template, compute similarity/distance metric e.g., cosine sim for vector rep.
6.  **Decision logic**
	- Have threshold guarding auth, compare score against threshold
	- Threshold tunes based on **usability, security level required, regulatory constraints, degree of feature extraction**.
7. **Outcome**
	- Success, fail, success may be fradulent, may lockout after repeated attempts etc. 

### Error tradeoffs
- **False acceptance rate** (FAR)
	- Attacker wrongly accepted
- **False rejection rate** (FRR)
	- Legit user wrongly rejected
- **Equal error rate**
	- Point where FAR = FRR
	![](Pasted%20image%2020260212212845.png)
	Not necessarily optimal operational setting; just comparitive metric. Threshold is key factor in this. FAR and FRR depend on sec requirements. For consumer, FAR acceptable, FRR minimised. Other way round in high security contexts.

### Biometrics security consideration
- No risk of losing access; inherent
- Convenient where system **well-designed**
- Availability of sensors
- Non-revocability
	- If templates leak, cannot change fingerprints
	- Fix: **canceallable biometrics** - distort biometric data before storage via non-invertible transformations, can cancel compromised ones, and issue new ones by changing the transformation parameters. 
- FAR vs FRR
- Ethical concerns
- Not foolproof
	- **Adversarial ML attacks,** small perturbations to model input to guide decision boundary to determine threshold/decision boundary e.g., slighr audio modulation, pixel changes for facial recognition. 
	- Backdoors in training data; e.g., teach master key, poison the training. 


## Possession Factors
- Security credential based on something user physically owns/has
- Claimant, controls object, can be used for auth; physical
- Often combine with knowledge or inherence
### Examples
- Keys(literal physical keys)
- **Hardware tokens**
	- Physical device that holds a digital implementation that only they could have
	- E.g., OTP, security keys
	- **OTP**
		- OTP token and server sync to same clock, generate new code every x seconds
		- Prevent unauthorised access even if main password stolen
		- Server side storage = dangerous, manual entry can lead to social engineering
	- **Security keys**
		- Devices that plug into or tap against the verifying object; they store a secret key and perform cryptographic operations internally to transmit info and verify identity.
	- Strength of hardware tokens depends on cryptographic design and whether keys are extractable. 
- **Smart Cards**
	- Advanced hardware tokens with embedded microprocessors.
	- May contain embedded private keys; tamper resistant hardware, pin activated protection
	- Need card readers; deployment cost, loss/theft risk. 
- **Smartphones**
	- Often act as OTP generators; convenience. 
	- Additional factors include biometric unlock before even using the phone as a possession factor. 
	- Combine possession, biometrics, knowledge into one, but single point of failure. 

### Hardware tokens: FIDO/WebAuthn
- Open standards permitting websites to register and authenticate users via hardware tokens. 
- During enrolment, authenticaor generates key pair
	- Private key kept inside device
- Server stores public key with some metadata e.g., user handle, credential ID
- During auth
	1. **Server generates challenge**
		- Pseudorandom, unpredictable value unique per login attempt.
	2. **Browser sends challenge to authenticator**
		- Passes challenge, credential ID, website origin to device. 
	3. **Hardware token signs challenge**
		- Adds the website origin, credential ID to the response too. 
	4. **Server verifies signature with the public key, authenticating the user.**

### Hardware tokens: OTP vs Security Keys
- **OTP**
	- Hardware, or digital(both really)
	- Time based, temporary code that changes
	- Prevent unauth access even if main password stolen
	- Server side storage = dangerous, manual entry lead social engineering. 
- **Security keys**
	- Public key crypto, private on device, public on server.
	- Challenge response during authentication, automatic if paired with authenticating device. 
	- Resistant to phishing, can't reveal secret, as stored on hardware.


### Security consideration
 - Relies on strong keys + crypto
 - Theft = difficult to scale
 - Automatic methods such as hardware tokens that are cryptographically based (security keys) are often part of MFA. 
 - Physical loss/damage, affect usability, difficulty of use depending on technical literacy
 - Cost

## Knowledge factors
- Passwords, passphrases, PINs, security questions; assumption is that only legit user knows answer
- Knowledge factors are based on shared secrets:
	- The user knows a secret
	- Server stores representation of secret
	- Authentication succeeds if user proves knowledge of secret - symmetrical trust model. 
- Theoretical strength is tied to **entropy**
	- Measures unpredictability of secret in tandem with size of search space
		- Security questions designed such that answers yield high entropy
- In practice, human-generated secrets have low entropy, usually predictable regardless of search space size
	- Passowrd reuse, predictable passwords
	- Can be guessed, cracked, etc
- The dissemination of knowledge factors in authentication is due to the fact it strikes am balance btwtween convenience, cost, portability, etc. 
	- Universal, low cost, revocable 
	- VS scalable attack surface, phishable, social engineering, must be secure server side.

### Attacks against knowledge factors
- **Offline cracking**
- **Phishing**
	- User enters secret, fake site, secret then transferable; worse with reused passwords.
- **Credential stuffing**
	- Password reuse, breach on one site, can then be used on others.
- **Social engineering**
	- Effective for security questions, typically low entropy, can be guessed with small amounts of info e.g., getting location can lead to street name guesses.


### Password managers
- Generate high-entropy poasswords, avoid reuse, store securely in encrypted vault. 
- Aim to make credential stuffing infeasible and increase entropy making offline cracking and other attack vectors less feasible
- **Large risk:**
	- **Master password compromise**
	- **Malware on host**
	- **Impact of vault breach catastrophic**
	- **Third party trust needed**