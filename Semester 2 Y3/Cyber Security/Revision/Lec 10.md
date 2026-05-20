### TCP/IP network stack
- 4 layer practical model(in-line with protocol implementations)
- Acts as framework for communication over network; guarantee data delivery
	- **Layer 4: Application layer**
		- Where user-facing protocols live e.g., HTTP/S. FTP, SMTP, DNS, define valid reqs, responses etc.
	- **Layer 3: Transport Layer**
		- Provides end-to-end comms between **specific processes** on two separate hosts
		- Handles port nums, segmentation of data into appropriate chunks
		- TCP for example; ordered, reliable delivery with error checking+correction, data buffered to prevent overflow, and resending with exponential backoff
		- UDP = connectionless, datagrams with no handshakes, decreased latency
		- Adds TCP header w/ source port, destination port, sequence number, ACK number, flags, etc
			![](Pasted%20image%2020260307230636.png)
	- **Layer 2: Internet Layer**
		- Layer responsible for logical addressing and routing - getting packets from host to destination across multiple networks. 
		- Handles IP addressing - numbers that identify hosts
		- Routers examine destination IPs and forward packets toward destination based on routing tables.
		- Large packets broken into smaller pieces from above, according to MTU
		- TTL
	- **Layer 1: Data-link Layer**
		- Handles how bits are physically encoded and sent over a medium e.g., copper wire, fibre
		- Packages packets into frames with a defined structure:
			![](Pasted%20image%2020260307230135.png)
			MAC addressing - every NIC has 48 bit unique MAC address burned in at manufacture.
		- **Error detection at link level - CRC checksums**
		- Ethernet = dominant wired LAN technology defining frame structure, error checking etc. 


### TCP/IP nesting headers
- Each protocol carries the protocol in the layer above it by appending headers which are subsequently unwrapped at the appropriate stage. 
	![](Pasted%20image%2020260307231039.png)


### IP security
- IP is **connectionless** - each packet is completely independent unit
- IP is **stateless** - routers and protocols maintain no memory of packets seen. 
- Provides best effort service:
	- Packets can be dropped, duplicated, reordered, or corrupted
	- TCP compensates at the trasnport layer by guaranteeing resending/delivery and segment numbering(guarantee order)
	- But IP, no way to determine whether packet has been tampered with in transit
		- The attacker can modify the contents via an on-path device and replace the checksum in the TCP header before rewrapping with their own frame
- IPv4 has no guaranteed security support
	- Was designed without built-in mechanisms for encryption, authentication, or integrity
	- IPv6 has guaranteed security support. 


### IPSec
- Optional in IPv4, mandatory in IPv6
- It is a suite of protocols providing security for IP packets at the IP layer. 
- Two security measures:
	- **IP Authentication Header**
		- Authentication only
		- Computes hash over packet contents and includes in header
		- Receiver recomputes
		- Less used, packet contents may change in transit e.g., TTL header, this also hashes some parts of header.
	- **IP Encapsulation Security Payload**
		- Practical mechanism which encrypts the payload, then authenticates that encrypted data.
- No mechanism against traffic analysis e.g., amount of packets, source, destination, all still visible
	- Need traffic obfuscation + onion routing. 


#### IP Encapsulation Security Payload
- Includes an additional header within IP packet describing what encryption and authentication are in use
	![](Pasted%20image%2020260307232526.png)
- **Security parameter index**
	- Lookup key - both parties use to lookup **Security Association**
		- Details encryption algoritm, authentication algorithm, SA lifetime
	- SA established by the **Internet Security Association and Key Management Protocol (ISAKMP)** during **Internet Key Exchange** handshake using Diffie-Hellman
		- Over IKE channel, negotiate cryptographic params, store in on-device Security Association database, then use these for communication. 
	- When a packet is received, look at SPI value in ESP header, find matching SAD entry, and decrypt and verify. 
- **Sequence number**
	- Begins at 1, increments by 1 for every packet sent under given SA
	- Without one, replay attack possible
		- Attacker capture ESP packet
		- Attacker later retransmits same captured packet, receiver process as normal.
	- With one:
		- Receiver maintains:
			- Highest sequence number seen
			- Sliding window
		- Accept N-31 packets, accommodates legit reordering, but packet with sequence number far behind is either or replay or far too delayed.

#### Encapsulation Security Payload (ESP) in transport mode
- In transport mode, the ESP protects the transport and application layer content. 
- Leaves the IP header intact:
	![](Pasted%20image%2020260307235023.png)
- IP header is outside ESP encryption + authentication:
	- TCP header and application data cannot be read or altered reliably - cannot determine what is being communicated, or services being used
	- IP - source IP, dest IP, packet sizes, all visible, know which machines are talking
	- No stopping traffic analysis or crude manipulation of IP header contents.

### ESP in Tunnel mode
- In tunnel mode, ESP protects the internet, transport, and application layer
- Treats IP packet as payload for a brand new IP packet
- Original IP header is now encrypted.
	- Traffic analysis is now severely limited, only know 2 gateways are communicating. But if those gateways server thousands of internal users, can't determine which users are communicating with which external services. 
- Key component of tunnel mode = gateway
	- Device that connets two different networks or forwards traffic between them
	- Sits at boundary between private network and public internet, performs IPSec operations (may be router, may be software). 
- Client will have modified local routing table, such that regular packets generated are intercepted by virtual interface
- This local gateway takes the original packet and encrypts, authenticates according to SA. 
- The new IP header has the source as the real public IP and the destination as the remote gateway public IP, who will de-encrypt the packet and then forward it to the correct machine. 


### VPNs
- A VPN is a system that uses tunnelling, typically IPSec, to make two networks behave like they are directly connected over the internet.
- The private comes from IPsec
- Device will install virtual network adapter and new routing rules
	- Will reroute all packets, or selected traffic to VPN interface
	- This will perform IPSec actions on outgoing IP packets; encrypting, authenticating, and wrapping new IP header with true source IP and remote gateway IP
	- A gateway on your side also receives encrypted tunnel packets and decrypts + authenticates them using the SA agreed suite.
- Can be a security bottleneck, particularly the gateways, so it is important to be holistic and vigilant. 
- Instrumental for remote working as they effectively extend the private network by providing authenticated access to resources via 2 gateways. 



# Network attacks
## Address Resolution Protocol
- Protocol used to discover link-layer address such as MAC address associated with internet layer address
- Operates locally. 
- Prior to transmitting IP packet, sending host consults **ARP Cache** for an existing IP->MAC binding
- On Cache miss, issues ARP IP request as ethernet broadcast, containing the target IP, received by all hosts on local broadcast domain. 
- Host with that IP responds with unicast ARP reply, containing MAC address, which requester caches with TTL, then constructs Ethernet frame. 
- For off-subnet UP destination, sending host resolves MAC of its default gateway rather than final destination; stored on device.
	- Gateway+subsequent router hpts are just router-to-router connects that form their own network sections, perform ARP, unwrap and wrap ethernet frames until reach destination IP. 
- When reach final dest, receive frame, see that destination MAC matches own MAC, destination IP matches IP on local network, so ARPs for that MAC address, then serialises, and is received. 


### ARP Cache Poisoning
- Exploit two weaknesses:
	- **No Authentication**
		- ARP replies carry no cryptographic proof that the sender legitimately owns the IP address they're claiming to have the MAC for.
	- **Unsolicited replies are accepted**
		- Machines update ARP cache upon receiving a reply, even if a request was never sent. Can't distinguish fabricated replies. 
- Attacker positions themselves between a router and victim. 
- Attacker sends two unsolicited fake ARP replies, faking IP address of both router and victim, sending them correspondingly. 
	- Attaches attacker MAC address to both replies, so all traffic goes through attacker.
	- MITM - inspect incoming and outgoing. 
- Attacker enables IP forwarding, so all packets are forwarded to real destinations after inspection; communication works normally from both parties' perspective. 
- ARP cache entires expire; resend fake replies. 5


### Arp Cache Poisoning protection
- **Static ARP entries**
	- Configure ARP entires for critical devices
	- E.g., default gateway
	- Good for small number of critical addresses; cannot be overwritten by random ARP replies
	- Poisoning gateway is most common attack vector
- Some OSs ignore unsoliciticted ARP replies by default.
- **ARPwatch**
	- Monitor ARP traffic on network segment
	- Maintain database of known IP->MAC mappings, alert on anomalies
		- Alters triggered by IP changing MAC (could be legit, or could be poisoning)
		- New MAC address appearing on network segment. 
		- Unsolicited ARP reply received
		- MAC address claiming multiple IPs

## DNS
- Application layer protocol, facilitates communication by translating human-readable domain names into IP addresses. 
- DNS uses UDP on port 53, speed.
	- DNS exchange - single question and answer, 3 way handshake and teardown would be more expensive than the query itself
	- UDP is stateless and connectionless - no session, no sequence, no verification that a response corresponds to a request. 
	- Any machine on network can send UDP packet claiming to be DNS response. 

### DNS Resolution + Caching
- When we query `google.co.uk` machine asks local resolver, typically router or ISP DNS resolver for corresponding IP
- Resolver either returns cached answer, or queries upstream to authoritative nameserver for domain
- DHCP; usually maintain DNS resolver IP locally and send all DNS queries to that resolver
- DNS queries upstream are all brand new independent queries. 
	![](Pasted%20image%2020260308175659.png)
	may look like this, with resolver 
- The final result is cached by the local resolver, and potentially on-device
- Successfully poisoned cache entries will thus affect users until they expire. 

### DNS Spoofing
- DNS spoofing corrupts the mapping between a domain name and a legit IP
- Victim's resolver returns attacker IP; connect to this instead.
- **METHOD ONE: CACHE POISONING**
	- UDP, no session, no handshake
	- When response returned, resolver checks only two things:
		- Does destination port match where query sent from?
		- Does the transaction ID in the response match the one in the query?
	- Turns DNS cache poisoning into a race condition - attacker must send matching response to resolver first (as it will discard others), so it is cached for TTL.
	- Attacker usually on different network. Port usually 53, but must cycle through all 65536 transaction IDs, and forge rest of the response as they see fit. 
		- Can usually send all responses within milliseconds. 
- **METHOD TWO: MAN IN THE MIDDLE**
	- Doesn't need to guess, can see traffic
	- Establish MITM via ARP cache poisoning
	- DNS queries are just UDP and plaintext with no encryption, forge DNS response according to examined DNS request.

### DNS spoofing protection
- DNSSEC adds cryptographic authenticity to DNS so forged responses are defeated
	- Inconsistent across internet
- DNS over TLS wraps DNS in a TLS session (DoT), DNS over HTTPs (DoH), both prevents eavesdropping and spooofing attaks are impossible as there is no way to see transaction id or source port
- Source port randomisation
- Hardening DNS server - open resolvers are DNS servers that respond to queries/responses from any IP rather than only intended clients


## TCP Sequence prediction attack
- TCP provides ordered delivery, over IPs unreliable unordered service
- **Sequence numbers** - data in TCP stream has a sequence number, each side track what sent and received. 
- Three way handshake establishes initial sequence numbers
- Server maintains window of acceptable sequence numbers it will accept, reject outside of this (replay, duplicate, too far behind)
- **Attack**
	- If attacker can determine/predict current sequence number, can inject packets server accepts as legit, will be accepted by window size
	- Need victim IP, server IP (only possible without tunnel-mode sec), source port of victim, current sequence numbers
	- Can learn sequence numbers, then DOS victim, inject forged packets
### Defense
- Randomised sequence numbers, Firewall config, IPSec