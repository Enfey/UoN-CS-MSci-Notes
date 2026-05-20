## Network segmentation
- Practice of dividing computer networks into smaller, isolated sub-networks
- Segmentation can be physical or logical:
- **Virtual Local Area Network(VLAN)**
	- Operates at Layer 2 of OSI model
	- Single physical switch can host multiple VLANs, with devices on different VLANs being unable to communicate directly, they must go through router, and optionally, network-level firewall at router
		- This is opposed to having no intermediary; communicating via the learned MAC address-port mapping the switch maintains, instead forward to router if not on same VLAN.
		- From perspective of devices on different VLANs, may as well be on entirely separate networks. 
- **Software Defined Network(SDN)**
	- Modern modern approach
	- **Control plane** - logic deciding where traffic goes
	- **Data plane** - hardware forwarding packets outside network
	- These are separated
	- In conventional network, every switch and router = self-contained, maintain own forwarding decisions/tables
	- If want to change network-level security policy, must apply to each gateway.
	- In SDN, control plane placed into software application called **SDN CONTROLLER**, centralising logic deciding where traffic goes
	- Via protocols, allow SDN controller to program switch forwarding tables
	- Via the upstream API, intrusion detection can instruct controllers to quaratine devices that are threatening, isolating in dedicated VLAn and dropping all traffic. 


### Lateral movement
- Term used to describe attacker spreading through network after initial foothold
- Encapsulates, number of techniques, navigate network, reach HVT.
- A **Flat Network** is a domain in which all servers + devices communicate directly, without restrictive firewalls
	- Lower admin overhead, but high capability for lateral movement; if compromise one device, can access entire network
- Almost every lateral movement technique or exploit requires the attacker controlled machine to be able to initiate a network request to the target. 
	- A segmented network localises the impact of network compromise by constraining lateral movement. 
	- Say the office VLAN and the database VLAN are on the same switch, the only traffic that crosses is that permitted by network-level firewall on router. Cannot send packet unless packet filtering permits. Could not even be a router, could reach application gateway first etc.
- Security is layered within private networks, by making networks logically distinct, you gain the security benefits of distinct networks without a large increase in operational strain

### Zero Trust Architecture
- In the past, internals of a network were considered trustworthy
- Security efforts focus on perimeter
- Authentication often happens at edge; when inside, generally free to communicate with internal resources easily
- Threats can originate from inside; lateral movement, sabotage etc
- Implicit trust broke down
	- Not only that, but structural reasons e.g., BYOD, network perimeter has become increasingly difficult to define
- Zero trust is response to these failings, ***never trust, always verify***
- **Explicit and mutual authentication**, **network segmentation,** **RBAC(**scoped as narrowly as possible e.g., role = engineer, origin = VLAN10, can access service X on VLAN12 on port 443 between 0700and2200), comprehensive logging and continuous monitoring. 


### WiFi Security
- Fundamental challenge = physical medium, radio waves, broadcast omnidirectionally, cannot be constrained to authorised recipients only. 
- Anyone in range, with appropriate hardware, can receive all frames
- Cryptographic security of protocols is the primary defense


### WiFi Security protocols
- **WEP** - Wired Equivalent Privacy
	- Used stream cipher w/ 40 bit key XOR'd with plaintext
	- Has IV, 24 bit, but only allowed 16.4 million distinct IVs
	- Caused IV reuse on same networks
	- Produce same keystream, can XOR to cancel keystream and yield XOR of plaintexts, can then recover
- **WPA** - WiFi protected Access
	- Patched WEPs worst flaws whilst remaining compatible with WEP hardware
	- Prevents IV reuse
	- **WPA2** leverages AES in in counter mode with CBC-MAC for integrity
		- Mostly sound cryptographic construction
		- When connecting WPA2 network, client must prove know password, and client and AP must end up with shared encryption key for session traffic. 
		- Derive key from wifi password, SSID using particular algorithm, applied a lot, to slow-down brute force attempts. 
		- 256 bit key derived, and is long-term, does not change unless password changes
		- Then send nonce from AP to client, client generates own nonce, feed everything into pseudorandom function.
		- Client sends its nonce, a HMAC computed over the message using a portion of the pairwise transient key, and some other stuff
		- AP derives same PTK in next step, computes same, verifies. 
		- However, offline dictionary attack is possible
			- Nonces, MAC addresses, message integrity codes are public
			- So one can brute force a PMK based on passwords, then take that and compute PTK, and compute MIC
			- Once the MIC matches, know correct
			- 4096 iterations for PMK designed to slow this down
	- WPA3 is in deployment and is most secure but not all devices yet compatible. 

### Common threats
- Build rainbow tables for popular SSIDs for WPA2 networks that are susceptible to offline dictionary attacks.
- Packet sniffing/eavesdropping enables attacks on improperly secured networks; can also decrypt retroactively after getting key. 
- **Rogue AP/Evil Twin/Karma**
	- Family of attacks exploiting the fact that 802.11 has no mechanism for clients to auuthenticate access points
	- Trust is entirely based on SSID string and signal strength. 
	- A **rogue AP** is an unauthorised WAP connecting to or operating near a network. 
		- May serve malicious **captive portal** - harvest credentials, by serving replica of login page.
	- **Evil twin** is a targeted external attack where an attacker clones existing SSIDs, and BSSIDs (MAC address of AP) to impersonate, so can inspect all plaintext traffic, inject malicious content
		- Set up with stronger transmit power, clients see as stronger signal
		- To accelerate process, attacker sends 802.11 deauth frames to connected clients - can forge by spoofing BSSID, makes them reconnect, and hopefully, to the malicious AP. 
	- **KARMA**
		- Exploits the probe request of client side 802.11 compliant devices
		- When client not connected to network, actively search for previously connected networks via **probe requests** - frame sent on each channel for SSID
		- Broadcast public
		- KARMA-device lists for probe requests, responds affirmatively to all, regardless of SSID
		- Devices connect silently with no user-interaction required.


### Good practices to nullify threats
- WPA3/WPA2 enterprise, not WPA2-PSK to avoid offline dictionary attack
- Disable WPS
	- Allows devices to connect via 8 digit pin or pressing physical button
	- Had design flaw - router validates pin in two halves, last digit = checksum
	- Therefore search space was actually 10^4 + 10^3, easily cracked
	- Also usually didn't have lockout
- Update firmware; use strong auth
- **MAC FILTERING** - whitelist permitted MAC addresses
	- Weak, MAC can be spoofed
	- MAC transmitted in plaintext too
	- Also MAC randomisation, hard to filter for this modern approach. 
- Network segmentation
- Detection systems - deauth frame flooding, rogue APs
	- Usually via WIPS typically have sensors, monitor all channels, send to central management system.

## Denial of Service
- A **denial of service attack** targets availability pillar of **CIA triad**
- Unlike attacks that aim to exfiltrate data or gain unauthorised access, DoS sole objective; make system, service, resource, unavailable.
- The general mechanism: **Resource Exhaustion**
	- Every system has finite resources; when one is exhausted, the system stops functioning.
- DoS finds the cheapest resource to exhaust from attacker's perspective, relative to what it costs the target to handle.
- A **distributed denial of service (DDoS)** occurs where there are multiple attacking machines.
	- This matters for two reasons:
		1. Massively increases attack capability beyond what a single machine can generate
		2. Makes mitigation far harder; traffic arrives from thousands of different source addresses, potentially via different vectors.

### TCP SYN flooding
- Resource exhaustion attack targeting TCP connection state.
	- When SYN arrives to initiate connection, server doesn't know if handshake will be completed.
	- It assumes it will, allocates half-open transmission control block (TCH), which records the state of this half-open connection (source/dest ip, ports, seq numbers)
	- It then sends SYN-ACK and waits; half-open TCB must be held until either ACK arrives(TCB then becomes established), or timeout expires
	- Half-open TCBs are held in an **incomplete connection queue**
- Attack sends large volume of SYN packets, typically with spoofed source IPs so the SYN-ACK responses go to random addresses
- Fills up the incomplete connection queue the kernel maintains, legit connections are dropped
- Entries held for 75 seconds, keep flooding, legit clients cant connect.


### Amplification attacks
- DoS attacks = bandwidth contents
	- If target has more bandwidth than attacker, attack fails
- This symmetry is broken by amplification attacks, where attackers exploit **protocol asymmetry** 
	- Find protocols where small request generates a disproportionately large response
	- Combine with IP spoofing, redirect amplified response at victim
	- Often use UDP, stateless sends to source IP with no handshake.
#### Smurf and Fraggle
- **Smurf**
	- Uses ICMP echo requests (pings)
	- Attacker sends ping to network broadcast address(IP that delivers packets to hosts on that subnet), with the victim's IP spoofed as the source.
	- Every host on that network sends ICMP echo replies to the source IP of the ICMP echo request, spoofed as the broadcast address usually
	- Takes down the broadcaster, network down. 
	- Amplification factor = linear in number of hosts in subnet
	![](Pasted%20image%2020260321203800.png)
- **Fraggle**
	- Same thing, but uses UDP echo packets, as it sends response to whatever source IP appears in packet header.
	- Same amplification factor

#### DNS amplification attacks
- Modern variant of amplification attacks
- Recursive resolvers respond to DNS queries, ask ISP, ISP asks other name servers, until get authoritative nameserver and returns.
- A 64 byte ANY query for gov.uk produces a 998 byte response, roughly a 15x amplification factor.
- With DNSSEC these responses are even larger due to cryptographic signatures being appended.
- Attack targets open resolvers configured to answer queries from any source IP:
	- Attacker queries open resolver with victim IP spoofed, and the resolver will send large responses to the victimn
	- Botnets often maintain curated list of known open resolvers
	- Projects like the open resolver project identify and notify operators of misconfigured resolvers

#### mitigations
- Rate limiting - limit rate at which near-identical responses are given to destination IP at resolver level
- Restricting resolver access by IPs - ISP resolvers e.g., only allow from ISPs customers.

### NTP amplification
- Protocol designed for synchronising clocks across networked machines to a common time reference 
- Accurate time = important e.g., logging, certificate validity
- Older NTP implementations included a monitoring command called `MON_GETLIST` intended for admins to query which clients had recently contacted the server, returns list of last 600 hosts to query server
- 200x amplification factor - mechanism = same, small spoofed UDP request, enormous response directed at victim. 

#### NTP amplification