# Malware
> Software that is intentionally designed to cause disruption to a computer, client, server, network. Leaks info, gains unauthorised access, deprives access to info, interfers with the computer's security and privacy.


## Categorising Malware
Usually categorised based on two properties.
- **How it proliferates**
	- Defines how the initial compromise occurs + how lateral movement happens
		- Is user interaction required?
	- Some malware follow exponential infection curves governed by variables like network toplogy + scanning strategy
- **What it does (payload)**
	- Determines attacker intent; numerous categories:
		- **Espionage**
			- Data exfiltration
		- **Monetisation**
			- Ransomware, mining
		- **Control**
			- BOTNET
		- **Destruction**
			- Sabotage
		- **Persistence**
			- Rootkits, backdoors


## Vectors
> An **infection vector** is the mechanism via which malware is able to initially infect the machine
- Common vectors:
	- **Software vulnerabilities:**
		- Use after free, buffer overflow, TOCTTOU logic bugs, config weaknesses
	- **Social engineering**

## Payloads
- **Malicious component deposited onto the machine**
- Range in severity
	- Nothing, just sit and wait
	- Messages and adverts
	- Botnet
	- Exfiltration
	- Destruction

## Virus
> A virus is malware usually hidden inside another *clean*/harmless program; it is a piece of code.
- When the user opens or runs the compromised binary, the virus runs, and attempts to proliferate to other files or computers.
- Requires a host program, and a consequent user action to run.
- Types of infection:
	- **Appending**
		- Appended to executable
		- Entry point modified to jump to virus
	- **Prepending**
		- Virus placed before original code.
	- **Cavity infection**
		- Injects into unused space within the on-disk binary e.g., alignment gaps, adjusts jumps and such accordingly.
		- Tedious, but difficult to detect; if small enough, may not even alter binary size.
- Advanced virus techniques include rewriting own code during proliferation, changing instruction order, and using equivalent instruction substitution to avoid detection. 
- Usual payloads: keylogging, file corruption, ransomware


## Worms
> A **worm** is a piece of self-replicating malware that spreads across networks automatically without human interaction
- Worms transmit themselves over a network to infect other computers and can copy themselves without infecting files(memory), spreading itself.
- **Propagation**:
	- **Random scanning**
		- Each infected host, generates, random IPs, sends exploit payload to that address
		- If vulnerable to the attack vector, becomes infected
		- Easily detectable, wastes time scanning non-existent IPs
	- **Hit-list scanning**
		- Worm begins with precompiled list of known vulnerable systems, prior recon
		- Infect hit-list first, disseminate parts of hit-list to infected host, who keep going and going, until exhausted
		- Then use random scanning
	- **Topological scanning**
		- Instead of scanning IP space, worm proliferates via relationship graphs
		- After infection, worm extracts info that would assist in proliferation e.g., email address books
		- Then sends itself to those contacts
		- This however, changes the vector; the initial infection vector and propagation vector may be different. 
			- But may not require human interaction e.g., exploit vulnerabilities in mail clients themselves.
- Usual payloads: botnet, DoS, backdoor installation, info harvesting


### Exploit Life Cycle
1. **Vulnerability Exists**
	- The bug/vulnerability is present in software and nobody publicly knowns about it.
	- If attackers discover, becomes **zero-day exploit**
2. **Disclosure**
	- Notify users that patch has been released/is to be released, fixing a bug
3. **Patch released**
	- When a patch is released, reverse engineers compare patched vs unpatched versions, identify changed functions + symbols
	- Determine what was vulnerable and reconstruct exploit.
	- Many exploits are reverse engineered from patches; full disclosure of exploit, not safe usually, cannot deploy patch to every machine instantly. 
4. **Exploit**
	- Once exploit is known, become integrated into malware
	- Many real-world exploitation, uses $n$ day exploits (recently patched but not yet deployed)

### Zero-Day Exploits
- A zero-day is a vulnerability unknown to the vendor, that would facilitate an attack
- Most dangerous form of exploit; no patch; bypass AV
- Valuable, used by nation states, rarely seen. 


## Trojans
> A trojan is a form of malware that masquerades as a legit program, persuading the victim to install it. 

- Carries payload that is deposited when executed. 
- Often proliferated via social engineering e.g., user duped into executing email attachment designed to be unsuspicious
- Payload can be anything. 
- Different to virus - virus attaches to existing files, fits into them. This  is a crafted executable with malicious intent, and as such are usually highly targeted.
	- They do not self-replicate or attach to other files. 
- Ransomware is the most common form of payload that a trojan contains, but may be backdoor payload, silent exfiltration, dropper to install rootkit from C&C server.
## Ransomware
> Ransomware is a type of malware that enforces denial of service, by locking victims out of access to their system or files.

- Then demand ransom
- AV removal of payload often too late - harm is in transformation of data, recovery needs key.
- Generally generate a random symmetric key with a particular mode, usually AES, each file may get its own nonce.
- Further encrypt AES key with attacker's public key, and then only attacker's private key can decrypt.
	- This protection is usually done via a command and control server, deliver unique KP per victim.
- **Crypto ransomware** - encrypts files, strongest leverage
- **Locker ransomware** - blocks access, locks screen blocks input, changes registry settings - easier to remediate
- Hardest component, tricking user into running it, and bypasing AV and browser protection
	- Most dangerous proliferate as worm e.g., wannacry
	- Malicious webpages
	- Trojans
	- Phishing
		- Work best with advance knowledge of victim


OFTEN IN PRACTICE, MALWARE IS A BLENDED THREAT AND IS DOES NOT STRICTLY SIT WITHIN ONE CATEGORY.

