## TCP/IP network stack
- 4 layer practical model (unlike OSI which is 7 layers and generally quite theoretical, this is in-line with protocol implementations).
- Acts as a framework for communication over a computer network with guarantee of data delivery
	- **Layer 4: Application Layer**
		- Topmost layer where user-facing protocols live, e.g, DNS, SMTP, FTP, SSH, HTTP/S defining valid requests and responses and how errors are communicated. 
	- **Layer 3: Transport Layer**
		- Provides end-to-end communication between specific processes on two separate hosts.
		- Handles port numbers, segmentation of data into appropriately sized chunks.
		- TCP for example, ordered, reliable delivery with error checking, with buffered data sent preventing overflow, and backoff with network detection. 
		- UDP = connectionless, datagrams with no handshakes, decreases latency. 
		- Adds TCP header:
			![](Pasted%20image%2020260307230636.png)
	- **Layer 2: Internet Layer**
		- This layer is responsible for logical addressing and routing - getting packets from a source host to a destination host across *multiple networks*, potentially spanning the entire globe.
		- Handles IP addressing - numbers that identify hosts on a network (typically distinguish machines by MAC address described below)
		- Routers at this layer examine destination IPs and forward packets toward destination.
		- Large packets are broken into smaller pieces according to MTU
		- TTL examined by routers. 
	- **Layer 1: Network Access Layer/Link layer**
		- Handles how bits are physically encoded and sent over a medium e.g., copper wire, fibre, radio waves
		- Packages data into frames with a defined structure![](Pasted%20image%2020260307230135.png)
		- **MAC addressing**, every NIC has a globally unique 48-bit MAC address burned in at manufacture
		- **Error detection** at link level e.g., CRC checksums in ethernet frames.
		- **Ethernet** is the dominant wired LAN technology, defines frame structure, error checking, and MAC addressing. 

### TCP/IP Nesting Headers
- Each protocol carries the protocol in the layer above by appending headers which are subsequently 'unwrapped' at the appropriate stage post-serialisation to wire. 
	![](Pasted%20image%2020260307231039.png)

### IP Security
- IP is connectionless (each packet is a completely independent unit with no established channel before data flows) and stateless (routers and the protocol maintain no memory of packets seen before).
	- Provides a best effort service - packets can be silently dropped, duplicated, reordered or corrupted. TCP compensates at the transport layer somewhat by guaranteeing resending, but IP has no way to determine whether a packet has been tampered with in transit, as the checksum is public in headers, the attacker can modify the contents via an on-path device and replace the checksum such that it looks valid. 
	- No delivery guarantee (provided by TCP) and no order guarantee (provided by TCP). 
- IPv4 has no guaranteed security support - was originally designed without built-in mechanisms for encryption, authentication, integrity.
- IPv6 has guaranteed security support.
### IPSec 
- Optional in IPv4, mandatory in IPv6. 
- Suite of protocols that work to provide security for IP packets at the IP layer. 
- Two majority security measures
	- **IP Authentication Header** - authentication only (less used, AH authenticates parts of IP header but some header fields legitimately change in transit e.g., TTL)
		- Computes a hash over the packet contents and includes it in the header, the receiver recomputes the hash and compares.
	- **IP Encapsulation Security Payload**
		- Practical mechanism which encrypts the payload and then authenticates that encrypted data. 
- No mechanism against traffic analysis e.g., amount of packets, source, destination, all still visible, strong protection against traffic analysis = traffic obfuscation, and onion routing. 
#### Encapsulation Security Payload
- Includes an additional header within the IP packet that describes what encryption and authentication are in use.
	![](Pasted%20image%2020260307232526.png)
- **Security parameter Index**
	- Essentially a lookup key
	- Rather than including full cryptographic parameters in every packet (which would be expensive), the SPI is a 32-bit value that both parties use to look up an agreed Security Association - a record containing:
		- Which encryption algorithm to use
		- Which authentication algorithm to use
		- The SA lifetime.
	- The SA is established by the **Internet Security Association and Key Management Protocol** (ISAKMP) during **Internet Key Exchange** (IKE) handshake using Diffie-Hellman for key exchange.
		- Diffie-Hellman establishes a secure channel, often use this shared-secret to derive keys/IV material for cryptographic algorithms.
		- Over the IKE channel, negotiate cryptographic parameters, and store in the on-device Security Association Database(SAD). The index of the stored entry is the SPI. 
		- When a packet arrives, the receiver looks at the SPI value in the ESP header and finds the matching SAD entry and can decrypt + verify using the correct algorithms. 
- **Sequence Number**
	- Starts at 1, and increments by 1 for every packet sent under a given SA. 
	- Without one, a replay attack is possible:
		- Attacker captures legitimate ESP packet, which is processed normally by receiver. The attacker later retransmits the same captured packet, and the receiver will process as normal (ideally).
	- With one:
		- Receiver maintains: 
			- Highest sequence number seen: N
			- Sliding window: accepts N-31 through N
		- Sliding window (normally 32 or 64 packets wide) accommodates legitimate reordering - but a a packet with a sequence number far behind the window is either a replay or hopelessly delayed and dropped regardless. 
##### ESP in Transport Mode
- In transport mode, ESP protects the transport and application layer content but leaves the original IP header intact. 
	![](Pasted%20image%2020260307235023.png)
- The IP header is completely outside the ESP protection. This has some consequences:
	- The TCP header and all application data cannot be read or altered reliably (encrypted using AES-GCM) - cannot determine what's being communicated or which specific service is being used
	- The source IP, destination IP, packet sizes etc, are all visible, they know which machines are talking, but cannot read the contents of the inner packets. No stopping traffic analysis or crude manipulation of TTL and other IP header contents. 
##### ESP in Tunnel Mode
- In tunnel mode, ESP protects the internet, transport, and application layer; it treats the original IP packet as the payload for a brand new IP packet. 
- Everything from the original IP header is now encrypted inside the new outer packet. 
	- Traffic analysis is now severely limited, only know that two gateways are communicating, but if those gateways serve thousands of internal users, the attackers cant determine which users are communicating with which external services.
	- But packet sizes and timing patterns can still leak information even when endpoints are hidden. 
- A key component of tunnel mode is the gateway. A gateway is ant device that connects two different networks or forwards traffic between them. 
- In the context of IPSec, a gateway sits at the boundary between a private network and the public internet, and performs IPSec operations on behalf of the private network(may be router, may be software). Internal devices don't perform any IPSec calculations whatsoever. 
	- Client has a modified local routing table, such that regular packets generated by machines are intercepted by a virtual interface. 
	- Gateway takes the original packet and encrypts, authenticates according to ESP, and wraps in new IP packet, with the source as the real public IP, and the destination as the receiver gateway public IP. 
	- This outer packet is then sent via the real network adapter. 

### VPNs
- A virtual private network, is a way to provide a secure connection over an unsecure network. 
- Different technologies operate at different layers to provide VPN functionality.
	- IPSec ESP tunnel mode at network layer
- Instrumental for remote working as they effectively extend the private network by providing authenticated access to resources via 2 gateways. 
- Can be a security bottleneck, particularly the gateways, so it is important to be holistic and vigilant. 
- DO MORE OF THIS WHEN REVISING. 
### Network attacks
#### Address Resolution Protocol
- Address resolution protocol (ARP) is a protocol used for discovering the link layer address such as a MAC address associated with an internet layer address, typically IPv4 addresses. 
- ARP operates locally.
- Prior to transmitting an IP packet, the sending host consults its local ARP cache for an existing IP->MAC binding, on a cache miss, it issues an ARP IP request as an ethernet broadcast (destination `FF:FF:FF:FF:FF:FF`) containing the target IP, which is received by all hosts on the local broadcast domain.
- The host owning that IP, responds with a unicast ARP reply containing its MAC address, which the requester caches with an associating TTL before constructing the Ethernet frame. 
- For off-subnet IP destinations the sending host resolves the MAC address of its default gateway rather than the final destination; this is known via the default gateway usually configured on device. The gateway (and subsequent router, as hops are just router-to-router connections that are their own network segments) perform ARP resolution and wrap the layer 2 and above contents back up in a new ethernet frame. 
	![](Pasted%20image%2020260308023014.png)
- When reaches final destination, receives the frame, and sees that destination MAC matches own MAC address, and destination IP matches IP address on local network, so ARPs for that MAC address, and serialises over the wire, and is then accessible by application. 

#### ARP Cache Poisoning
- Exploit two weaknesses in the ARP protocol
	- **No Authentication** - ARP replies carry no cryptographic proof that the send legitimately owns the IP address they're claiming, any machine can send an ARP reply claiming any IP.
	- **Unsolicited replies are accepted** - a machine updates its ARP cache upon receiving a reply even if it never sent a corresponding request. There is no mechanism to distinguish legitimate replies from fabricated ones. 
- Attacker positions themselves between a router and a victim, intercepting all traffic between them. 
- Attacker sends two unsolicited fake ARP replies, faking the IP address (of both the router and the victim, and sends them correspondingly) but providing the attacker's MAC address, so all traffic flows through the attacker. 
	- The attacker enables IP forwarding so packets are still forwarded to real destinations after inspection, meaning communication still works normally from both parties' perspective.
		![](Pasted%20image%2020260308173939.png)
- ARP cache entries expire every few mins, must continously re-send fake replies. 

#### ARP Cache Poisoning protection
- Static ARP entries - manually configure ARP entries for critical devices such that they cannot be overwritten by any received ARP reply - legitimate or otherwise. Highly effective for a small number of critical addresses, particularly the default gateway.
	- This is because poisoning the gateway is the most common attack vector. 
	- Doesn't scale
- Some OSs ignore unsolicited ARP replies by default such as OpenBSD.
- Tools like **ARPwatch** monitor all ARP traffic on a network segment, maintain database of known IP->MAC mappings and alert on anomalies
	- Alerts triggered by:
		- IP changing MAC address(could be legitimate e.g., changing NIC, or could be poisoning attack)
		- New MAC address appearing on network segment
		- MAC address claiming multiple IPs
		- Unsolicited ARP reply received.

#### DNS
- Application layer protocol, facilitates communication by translating human-readable domain names into IP addresses. 
- DNS uses UDP on port 53 for speed usually
	- This is because a DNS exchange is a single question and single answer, typically under 512 bytes each, a 3 way handshake and teardown would add more overhead than the query itself. 
	- UDP is stateless and connectionless - there is no established session, no sequence numbers, no built-in verification that a response corresponds to a request etc.
	- Any machine on the network can send a UDP packet claiming to be a DNS response. 
- **Resolution + Caching**
	- When we query `google.co.uk` machine asks local resolver - typically router or ISP's DNS server. That resolver either returns a cached answer or queries upstream until it reaches the authoritative nameserver for that domain. 
		- The host has no knowledge of the authortative name server IP. DHCP configuration; usually maintain a DNS resolver IP, and sends all DNS queries to that resolver.
		- DNS queries upstream are all independent brand new queries e.g., 
			![](Pasted%20image%2020260308175659.png)
			Presuming no caching, and then fills `google.co.uk` in DNS cache of each resolver when returning. 
	- The response travels back down the chain, with each resolver caching it for the record's TTL duration (a successfully poisoned cache entry will affect users until it expires).
#### DNS spoofing
- DNS spoofing corrupts the mapping between a domain name and a legitimate IP. 
- The victim's resolver returns the attacker's IP instead of the real one.
- The victim then connects to the attacker server believing it's the legitimate destination.
- **Method one: Cache Poisoning**
	- UDP has no session, no handshake etc. 
	- When a response is returned, a resolver only checks two things to check whether a response is legitimate:
		- Does destination port match where I sent my query from?
		- Does the transaction ID in the response match the one in my query? (part of DNS protocol)
	- This turns DNS cache poisoning into a race condition - the attacker must send a matching response to the resolver first, and the resolver will cache it and discard subsequent responses. 
	- Attacker is on a different network, not in MiTM position. The port is usually 53, but must cycle through all 65536 transaction IDs, and then forge the rest of the request as they see fit. 
		- Can usually send all 65536 responses within milliseconds. 
- **Method two: Man-in-the-middle spoofing**
	- Doesn't need to guess anything, as they can see the traffic difrectly. 
	- Establish MiTM position through ARP cache poisoning. 
	- DNS queries are UDP and plaintext with no encryption whatsoever, so can just read the packet and forge the DNS request with no issues. 

#### DNS spoofing protection
- DNSSEC adds cryptographic authenticity to DNS, so that forged responses are defeated. 
	- But inconsistent across internet
- DNS over TLS wraps DNS in a TLS session (DoT), DNS over HTTPS (DoH), booth prevent eavesdropping, and spoofing attacks are impossible as there is no way to see the transaction ID or source port (see below). DoH makes DNS traffic indistinguishable from regular traffic. 
- Source port randomisation - makes forged responses harder to match.
- Hardening DNS server e.g., open resolvers are DNS servers that respond to queries/responses from any IP rather than only intended clients. 
#### TCP Sequence prediction attack
- TCP provides reliable ordered delivery over IP's unreliable unordered service.
- The mechanism that makes this possible is **sequence numbers** - each byte of data in a TCP stream has a sequence has a sequence number, and both sides track what they've sent and received. 
- The three way handshake establishes sequence numbers:
	1. SYN seq = 1000
	2. SYN-ack seq = 5000, ack 1001
		- Received byte 1000, send 1001 next
	3. ACK seq = 1001, ack = 5001.
	Server maintains window of acceptable sequence numbers it will accept, anything outside this window is rejected - either duplicate or replay. 
- Attack:
	- If an attacker can determine or predict the current sequence number of an existing TCP session between a victim and a server, they can inject packets the server accepts as legitimate, as will be accepted by the window size. 
	- Need victim IP, server IP (easily discovered without tunnel-mode IPSec), source port of victim, current sequence number.
	- Can learn sequence numbers (early implementations used predictable counters), DOS victim, and inject forged packets. 
#### Defense against TCP prediction attack
- Randomised sequence numbers
- Firewall configuration.
- IPSec, hide IP addresses and port numbers. 
- Not common, but still motivates safeguards. 
