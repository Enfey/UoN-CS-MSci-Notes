# What do we mean by security
- **Defining security**
	- What does it mean to be secure:
		- Unbreakable vs secure enough
		- Tradeoffs, risk assessment, cost/benefit
		- Often simply an arms race between developers/researchers and malicious actors
			- Reactive(protecting against worst attack yet)
			- Proactive (protecting against assumed threats)
- Usually implemented via three broad kinds of measures
	1. **Prevention**
		Stopping attacks from succeeding e.g., access controls, encryption
	2. **Detection**
		Intrusion detection, audit logs
	3. **Recovery**
		Backups, incident response plans
- Perfect security, does not exist in practice, unbreakable systems unrealistic, so we design to be **secure enough** given known and predicted threats.
## Managing security
- Within organisations, security is a **management responsibility**, not a developer one
	- **Management defines what must be protected, developers implement these requirements**
- A concise document formalising the requirements is a **security policy**, typically defining:
	- Assets
	- Threats
	- Acceptable use
	- Responsibilities
	- Legal and regulatory constraints e.g, GDPR.
- Govern everything from password rules to audit trails.
- Thus, security failures generally manifest as policy failures, not technical ones. 

## CIA Triad: Core Security Goals
- Most security objectives fall under the **CIA Triad**
	1. Confidentiality
		*Prevention of **unauthorised disclosure** of information*
		- Do not want to compromise secret/private info e.g., medical records, private confirmations, credit car details.
	2. Integrity
		*Prevention of **unauthorised modification** of information*
		- Bank balances, database records, config files, not just about stopping attackers, must detect accidental corruption, need assurances that such data remains unmodified.
	3. Availability
		*Prevention of **unauthorised withholding** of information or resources*
		- Accessible and usable upon demand, denial of service attacks, ransomware, depends on redundancy, fault tolerance, backups, availability issues often caused by non-malicious issues too like hardware faults or misconfiguration. 

### Authenticity
- Just because we have integrity, doesn't mean we have authenticity, the guarantee that the data is sound doesn't verify who sent it
- Authenticity = integrity + freshness
	- Freshness prevents **replay attacks** where old valid messages are resent e.g., duplicate bank transfers

### Accountability
- Users should be **responsible** for their actions
- System should identify and **authenticate** users and ensure compliance
- **Audit trails** must be kept
- Often has conflicting values with privacy e.g., significant logging for compliance + incident investigation

### Non-Repudiation
- Ensures someone **cannot deny** an action later; indisputable evidence that they committed the action
	- Digital signatures
	- Certificates
	- Cryptocurrencies
	- Keycards
- Mostly a legal concept; evidence verifiable by a trusted third party.

### Security vs Usability
- *Security-unaware users have specific security requirements but no security expertise*
- That is, security design often results in **maximisation**
- There is a trade-off between security and ease of use, strong security increases friction, and users have to work around controls, when those specifying requirements are inexperienced, usability suffers heavily.
	- Multi-factor authentication
	- Disk encryption
	- Timeouts and lockouts
	- Excessive CAPTCHA
- Historically, systems that prioritised usability sacrificed security, while more secure systems initially frustrated systems.
- **Thus, good security design is about compromise, not maximisation.** 

### Data vs Information
 - Security mechanisms usually protect data, not information
	 - Data = raw bits
	 - Information = meaning derived from the data
- This leads to **inference attacks**, where protected info can be revealed indirectly, even though the data is secure. 
- For example, probing a database, and it says "you do not have permission to access Joe's criminal record", reveals that it does exist, even if the data itself cannot be accessed, the information around it that can be determined may supply additional attack vectors.

### Tradeoffs of Computer Security
- Some Objectives are conflicting:
	- Security vs usability/availability (MFA, disk encryption, lockouts + timeouts, CAPTCHAs, E2E, data collection vs personalisation etc)
	- Accountability vs privacy e.g., audit trails
	- Overheard(computation, communication, storage) and costs(organisational + societal)
- Security decisions are therefore **context-dependent**, not absolute. 
## Security Design
### Security Focus
- Computer Security, should be designed in from the start
- Adding later, leads to brittle and incomplete solutions
- Must approach security in a systematic, disciplined, and well planned manner, from system inception/design.

### Security design
- Good security design focuses on these principles:
	- **Focus of control**
		- In a given application, should the focus of protection be:
			- Data?
			- Operations?
			- Users?
		- In practice, often mix all three e.g., file permissions for users, API restrictions for operations, and database constraints for data.
	- **Complexity vs assurance**
		- Simple systems easier to reason about and verify, feature-rich systems are harder to secure and reason about.
	- **Centralised or decentralised controls**
		- We ask: *should defining and enforcing security be performed by a central entity, or be left to individual components in a system*
			- Central entity may be a possible bottleneck, and create single point of failure.
			- Decentralised control scales better, but is harder to reason about/manage.
	- **Layered security.**
		- Can visualise security in layers
		- Each layer protects a boundary, and relies on the security of the layers below. 
			![](Pasted%20image%2020260204175924.png)
		- Consider an application using a database
			![](Pasted%20image%2020260204180006.png)
			Each layer assumes layer below is secure, but if cannot break at one layer, just attack the layer underneath it e.g., bypass application logic via direct database access, bypass OS protections via kernel bugs.
