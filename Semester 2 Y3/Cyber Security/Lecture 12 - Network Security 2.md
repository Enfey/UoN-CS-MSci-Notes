## Network Segmentation
- Practice of dividing computer networks, into smaller, isolated sub-networks
- Segmentation can be physical or logical:
	- **Virtual Local Area Networks (VLAN)**
		- Operates at Layer 2, the data-link layer, of the OSI model. 
		- A single physical switch can host multiple VLANs, with devices on different VLANs being unable to communicate directly - they must go through a router, and optionally a network-level firewall. 
			- This is opposed to having no intermediary; communicating directly via the learned MAC address - port mapping the switch maintains, instead forwards to router if not on the same VLAN. 
			- From the perspective of devices on different VLANs, may as well be on entirely separate switches. 
		- This is the most common segmentation technique. 
	- **Software-Defined networks(SDN)**
		- A more modern approach where the *control plane* (logic deciding where traffic goes) is separated from the *data plane* (the hardware that actually forwards packets outside the network. )
			- In a conventional network, every switch and router is self-contained, maintaining their own forwarding tables, making own decisions about where to send packets.
			- If want to change a network-level security policy e.g., block traffic between two departments, must log into each affected switch individually. 
				- Switch maintains MAC address to port mapping for forwarding decisions.
			- In an SDN architecture, control plane is lifted out of individual switches and placed into a software application called the SDN controller.
			- Centralises the logic deciding where traffic goes, via standardised protocols which allow the SDN controller to program switch forwarding tables.
				- Usually have some application layer software e.g., a monitoring tool, segmentation manager, that declaratively describe the network, and the SDN controller figures out how to make it happen by updating switches uniformly.
		- Can micro-segment, policy changes are instant and global. 
		- Via upstream API aka northbound API, intrusion detection can instruct controllers to quarantine devices that are threating, isolating it in a dedicated VLAN or dropping all its traffic. 
- Aim to constrain lateral movement (from both insider threats, and from external breaches), and contain damage.
- May further make policy management and enforcement simpler.

### Lateral Movement
- Lateral movement is the term for an attacker spreading through after their initial foothold in the network. 
- Encapsulates a number of techniques used to navigate through a network, escalate privileges, and reach high-value targets. 
- A flat network is a domain in which all servers and devices can communicate directly with each other without restrictive firewalls/segmentation
	- Whilst having lower administrative overhead, there is high capability for lateral movement, as if one low-level device is compromised, attackers can easily access the entire network. 
		- Almost every lateral movement technique or exploits require the attacker-controlled/accessed machine to be able to initiate a network connection to the target. 
- Alternatively, a segmented network localises impact of network compromise. 
	- It constrains lateral movement, making it so attacks do not scale as easily. 
	- When say, the office VLAN and the database VLAN are on the same switch, the only traffic that crosses between them is traffic permitted by the network-level firewall on the router, and device-firewalls. An attacker on a compromised machine cannot send a single packet unless the ruleset says that traffic is permitted.
	- Security is layered within even private networks; by making these networks logically distinct, you gain the security benefits of actually having distinct networks, without a large increase in operational strain. 
## Zero Trust Architecture
- In the past, the internals of a network were considered trustworthy
- Security efforts focused on protecting the network perimeter due to this implicit trust model. 
- Authentication often happens at the edge - when a user is inside the perimeter, they were generally free to communicate with internal resources with minimal further challenge. 
- As we now know, threats can very much originate from within the internal network
	- Sabotage, lone infected device w/lateral movement, disgruntled employee etc. 
- Thus, the implicit trust model broke down, not only due to that, but due to structural reasons - the network perimeter has become increasingly difficult to define with cloud-hosted services, remote access, BYOD. 
	- Inside and Outside no longer cleanly map onto trusted and untrusted. 
- Zero trust is a response to the above failings: "never trust, always verify" regardless of network origin.
- Explicit and mutual authentication, network segmentation, RBAC (scoped as narrowly as possible with many attributes e.g., role = engineer, origin = VLAN10 etc may access service X on port 443 between 0700-2200), comprehensive logging and continuous monitoring. 

## WiFi Security
- Fundamental challenge is the physical medium: radio waves are broadcast omnidirectionally and cannot be constrained to authorised recipients. 
- Anyone within range with appropriate hardware can receive every frame transmitted. 
- The cryptographic security of the protocols are the primary defence.

### WiFi security protocols
- **WEP** - Wired Equivalent Privacy. 
	- Used RC4, stream cipher, with 40-bit key, generate psuedorandom keystream XOR'd with plaintext. 
	- Has IV, 24 bit, but only allowed 16.7 million distinct IVs.
	- This would cause IV reuse on busy networks. 
	- Would produce the same keystream, and can XOR them to cancel keystream and yield XOR of plaintexts, and can then recover parts of, if not the full plaintext. 
	- Attacker could modify ciphertext predictably without knowing the key.
- **WPA** - WiFi Protected Access
	- Emergency standard, patched WEP's worst flaws whilst remaining compatible with existing WEP hardware. 
	- Preventing IV reuse exploitation. 
	- **WPA2** leverages AES in counter mode with CBC-MAC for integrity, and is a mostly sound cryptographic construction.
		- When connect to WPA2 network, client needs to prove that it knows the password, and both client and AP need to end up with shared encryption key for session's traffic. 
		- The PMK (Pairwise Master Key) is a 256 bit value derived deterministically from the WiFi password using PBKDF2
			`PMK = PBKDF2(HMAC-SHA1, passphrase, SSID, 4096 iterations, 256 bits)`
		- HMAC-SHA1 is a keyed hash, pseudorandom, passphrase is the wifi password, SSID is network name, used as a salt, preventing pre-computed lookup tables from working across networks. 
		- 4096 iterations, means applied 4096 times, to slow down brute-force attempts. 
		- The PMK is long-term and static, and does not change unless the password changes. 
		- First message from AP is a nonce, ANONCE sent in plaintext; we do not encrypt directly under the PMK for obvious reason. 
		- The client generates its own nonce, and feeds everything into a pseudorandom function:
			`PTK = PRF(PMK, "Pairwise key expansion", 
	        `min(AP_MAC, Client_MAC) || max(AP_MAC, Client_MAC) ||`min(ANonce, SNonce) || max(ANonce, `SNonce))
	    - Client sends SNONCE and MIC (Message Integrity Code) - a HMAC computed over the message using a portion of the just-derived PTK (in the next step, the AP derives the same PTK and computes this MIC, which can only be done if know the password, and thus have same PMK, so is a valid connection).
	    - There is some additional setup amongst this, but this all creates a pairwise transient key for that session. 
	    - However, this enables an offline dictionary attack. 
		    - The ANonce, SNONCE, MAC addresses, and MIC are all visible, (MAC from frame headers, the rest as payloads in unencrypted packets).
		    - So compute PMK based on candidate_password worklist, then take that and compute PTK, and then compute MIC.
		    - Once the MIC matches, know that the candidate passowrd is correct.
		    - 4096 iterations for the PMK is designed to slow down this brute-force; GPU parallelism nullifies some of this.
	- **WPA3** is in deployment, is the most secure but not all devices are yet compatible.

### Common Threats
- Can build rainbow tables for popular SSIDs for WPA2 networks that are susceptible to offline dictionary attacks. 
- Packet sniffing/eavesdropping enables attacks on improperly secured networks - it is trivial on unencrypted networks, and even on WPA2 if one can obtain the session keys, captured traffic can be decrypted retrospectively. 
- Rogue AP/Evil Twin/Karma
	- Family of attacks exploiting the fact that 802.11 has no mechanism for clients to authenticate access points. 
	- Trust is entirely based on SSID string and signal strength, both of which are forgeable.
	- A **rogue AP** is an unauthorised WAP connected to or operating near a network.
		- May serve malicious **captive portal** - harvest credentials used for say, hotel/airport login pages. 
		- Returns convincing replica, 
	- **Evil twin** is a targeted external attack where the attacker clones an existing networks SSID, and potentially its BSSID (MAC address of AP) to impersonate, so can inspect all plaintext traffic, inject malicious content etc. 
		- Set up with stronger transmit power so clients see it as the stronger signal
		- To accelerate the process, attacker often sends deauthentication frames to currently-connected clients - 802.11 management frames are completely unauthenticated in WPA2, so can forge a deauth frame spoofing the BSSID, telling clients they have been disconnected, and then they reconnect, hopefully to the malicious AP.
	- **KARMA** exploits the probe request behaviour of client-side 802.11 compatible devices
		- When an 802.11 client is not connected to any network, it actively searches for previously connected networks by broadcasting **probe requests** - with a frame sent on each channel for each SSID in the client's preferred network list. 
		- This broadcast is visible to everyone within radio range.
		- A **KARMA**-capable device listens for probe requests and responds affirmatively to all of them, regardless of SSID, impersonating the network. 
		- Effective, devices connect silently with no user-interaction required. 
- Password cracking, see above. 

### Good practices to nullify threats
- WPA3/WPA2 enterprise, not WPA2-PSK to avoid offline dictionary crack attack. 
- Disable WPS - allowed devices to connect by entering 8 digit pin, or pressing physical button
	- Had design flaw with pin, router validates PIN in two halves.
	- Last digit was checksum.
	- Search space was therefore not 10^8 = (100,000,000), but 10^4 + 10^3 = 11000
	- Easily cracked, also didn't usually have lockout so slow the attack.
	- Completely bypasses all security measures.
- Keep firmware updated
- Strong authentication
- MAC filtering - whitelist of permitted device MAC addresses.
	- Weak though, MAC is trivially spoofed.
	- MAC is transmitted in plaintext in every 802.11 frame so can easily be impersonated.
	- Also MAC address randomisation, hard to do this filtering.
- Network segmentation for legacy and IoT devices and BYOD devices.
- Detection systems, detect rogue APs, evil twin/KARMA, jamming, deuath frame flooding etc.
	- Usually via WIPS(wireless intrustion prevention) typically consisting of dedicated sensors that monitor all 802.11 channels.
	- Send this data to a central management system.

## Denial of Service
- A **Denial of Service** attack targets the availability pillar of the **CIA triad**. 
- Unlike attacks that aim to exfiltrate data or gain unauthorised access, a DoS attack's sole objective is to make a system, service, or network resource simply unavailable to its legitimate users
	- At least on the surface this is the case.
- The general mechanism behind this is **resource exhaustion**, every system has finite resources, when one is exhausted, the system stops functions normally.
- DoS attacks find the cheapest resource to exhaust from the attacker's perspective relative to what it costs the target to handle
- A **distributed denial of service** (DDoS) occurs where there are multiple attacking machines.
	- This matters for two reasons:
		1. Massively increases the attack capability beyond what any single machine can generate
		2. Makes mitigation far harder since traffic arrives from thousands of different source addresses.

### TCP SYN flooding
- Straightforward resource exhausting attack targeting the TCP connection state specifically
	- When SYN arrives to initiate connection, the server does not know whether the 3 way handshake will be completed
	- It assumes that it will and allocates a half-open transmission control block (TCB), which records the state of this half-open connection (source/destination IP, ports, sequence numbers).
	- It then sends SYN-ACK and waits, this half-open TCB must be held until either the ACK arrives completing the connection(at which point it becomes established), or the timeout expires. 
	- Half-open TCB's are held in an **incomplete connection queue**.
- The attack itself sends an enormous volume of SYN packets, typically with spoofed source IPs so the SYN-ACK responses go to random addresses and no ACK ever returns.
- This fills up the incomplete connection queue, which the kernel maintains, so new connections from legitimate clients are dropped because the queue is full. 
- Entries are held for around 75 seconds, just keep flooding on timer, client can then never get in. 

### Amplification attacks
- Regular DoS attacks are a direct bandwidth contest between the attacker and a target.
	- If the attacker has more bandwidth than the attacker, then the attack fails.
- Amplification attacks break this symmetry by exploiting **protocol asymmetry** - that is, finding protocols where a small request generates a disproportionately large response and then combining it with IP spoofing to redirect that amplified response at the victim. 
	- Often use UDP, as it sends response to whatever source IP appears in packet header with no handshake to verify that address is real. 
#### Smurf and Fraggle
- Older broadcast-based amplification attacks
- **Smurf**
	- Uses ICMP echo requests (pings)
	- The attacker sends a ping to a network's broadcast address - the IP that delivers the packet to every host on that subnet simultaneously, with the victim's IP spoofed as the source.
	- Every host on that network sends an ICMP echo reply to the source IP of the ICMP echo request, which is spoofed as the broadcast address, or the address of a victim. 
		![](Pasted%20image%2020260321203800.png)
	- The amplification factor is linear in the number of hosts in the broadcast domain. 
- **Fraggle**
	- Same thing, but uses UDP, as it sends response to whatever source IP appears in packet header. The amplification mechanism thus behaves the same. 
#### DNS amplification
- This is the modern variant of amplification attacks.
- Recursive resolvers respond to DNS queries, asks root server, then goes to TLDs, then to nameservers, and propagates result along resolvers into cache for local resolver, so don't have to walk the chain again. 
- A 64 byte ANY query for gov.uk produces a 998 byte response, roughly a 15x amplification factor.
- With DNSSEC, these responses are even larger due to cryptographic signatures being appended.
- The attack targets open resolvers configured to answer recursive queries from any source IP.
	- Attacker queries open resolvers with victim's IP spoofed, and each resolver sends its large response to the victim. 
	- Botnets often maintain a curated list of known open resolvers
	- Projects like the Open Resolver Project attempt to identify and notify operators of misconfigured servers

##### Mitigations
- Restricting resolver access by source IP - an ISP's resolver should only answer queries from that ISP's customers
- Response Rate Limiting - limits the rate at which near-identical responses are given to a given destination IP.
- Minimal responses. 

#### NTP amplification
- NTP is a protocol designed for synchronising clocks across networked machines to a common time reference. 
- Accurate time is important for security infrastructure e.g., TLS certificate validity windows, log correlation.
- Older NTP implementations included a monitoring command called `MON_GETLIST` intended for admins to query which clients had recently contacted the server, returning a list of the last 600 hosts to query the server.
- Had 200x amplification factor.
- Underlying mechanism is identical to DNS amplification - small spoofed UDP request, enormous response directed at the victim. 