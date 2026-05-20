## Firewalls
- Hardware/software system that sits at network boundaries
- Controls which packets are allowed to pass through them
	ALL TRAFFIC LEAVING SUBNET MUST PASS THROUGH FIREWALL

## Firewall Functions
- **Single Point Security**
	- Enforce policy at network boundary; everything inside is implicitly trusted, everything outside is implicitly untrusted
- **Packet analysis and logging**
	- Every packet can be inspected and logged, providing security event records and visibility into net activity
- **RBAC**
	- Define ruleset determining what is allowed vs what is blocked; define via poliicy, rule based. 

## Firewall Location
- **Network Firewalls**
	- Placed between subnet and internet
	- Home router, most familiar example sit between home network + ISP, block all unsolicited inbound connections by default, allow outbound
		- Enterprise do same, at much larger scale, more sophisticated rulesets
- **Host-based Firewalls**
	- Placed on individual machines, filter traffic at machine level
	- Defends against compromised machines on internal network, and attacker who bypass network firewall.
	- Never assume anything on internal is safe. 


### Demilitarised zone
- Architectural concept, small subnet that separates externally facing services from internal network
- Sits between internet and internal network, separated by network level firewalls on both sides
- Damage containment - attacker who compromises outer firewall and web server has to bypass another firewall.
	![](Pasted%20image%2020260308184053.png)
- More granular control of the diff firewalls; inner one could be be much more restrictive. 

### Firewall basic functions
- **Defend against external access**
	- Prevent internet parties from accessing internal services; without firewall, every service on every internal machine potentially reachable from internet. 
- **Restricting internal to external access**
	- Firewalls can control what internal users can cannoect to externally. 
	- E.g., compromised machine, don't want to allow to exfiltrate to random internet server. 
	- IRC historicaly used as CNC channel for botnets, IRC = outbound, so need filter. 
- **Network address translation**


### Firewalls are not sufficient on their own
- **Tunnelling**
	- If attacker can encapsulate malicious traffic inside a permitted protocol, firewall cannot distinguish. 
	- HTTPS on port 443 universally permitted, malware communicating over HTTPS looks identical to legit web traffic
	- DNS tunnelling encodes data in DNS queries which firewalls rarely block. 
- **Internal threats**
	- Only protect perimeter; controlling outbound + host firewalls can help, but if attacker has physical access or access to multiple machines, doesnt do much. 
- **Virus-infected files**
	- Firewall sees permitted HTTP connection download a file, outbound. It does not inspect the returned packets to see if they are corrupt or malformed - it has no idea what the correct contents should look like. 

### Packet filters
- Most fundamental firewall mechanism
- Examine individual packets and apply rules based on **header information only**
	- **Source IP**
	- **Destination IP**
	- **Protocol**
	- **Source port**
	- **Destination port**
	- etc
- Used for both inbound and outbound
- Operate at layers 2+3; zero knowledge of what application generates the traffic or what the data contains. 


### Packet filter rules
- Rule execution depends on implementation
	- IPtables: first rule to match is applied
	- PF: all rules eval, last matching one wins
- Every packet filter has a default action for packets that match no rule, which is the most important policy decision
	- Usually default ACCEPT or DROP
	- Easy to DOS onesself by blocking necessary traffic. 

### IPTables
- Userspace tool for configuring Linux's `netfilter` kernel firewall.

#### Tables and Chains
- IPTables organises rules into **four default tables**
	1. **Filter**
		- default table, used for allowing and blocking packets
	2. **NAT**
		- rewrites source and destination addresses
	3. **Mangle**
		- packet alteration e.g., TTL modification
	4. **Raw**
		- skips connection tracking, used for performance optimisation
- Within each table, rules organised into chains, which have ordering:
	- **INPUT CHAIN** - packets destined for this machine
	- **OUTPUT CHAIN** - packets generated from local machine
	- **FORWARD CHAIN** - all other packets passing through (gateway)
- Matches result via jumps, check next rule; inspect table, inspect chains with rules, jump to other chains, etc, applying the first rule that matches until policy decision is made regarding that packet:
	![](Pasted%20image%2020260519165504.png)

### Types of policies
- **Permissive**
	- Allow everything except for dangerous services
	- Fundamentally reactive
	- Unknown threats, forgotten services, newly installed, automatically permitted
	- DEFAULT = ACCEPT
- **Restrictive**
	- Blocks everything except designated useful services
	- More secure, but easy to DOS oneseelf
		- Default DROP, forget to allow DNS requests outbound, unable to resolve names. 
		- Debugging can be difficult.

## Packet Filter issues
- Simple, low level, high assurance and ability to reason about
- **Cannot prevent against attacks that employ application specific vulnerabilities**
	- No idea whether packet payload is legit or is malicious itself
- **Easy to access or deny packets incorrectly**
	- Rule ordering is critical, single misplaced rule, facilitate attacks or break legit traffic. 
- **No higher-level authentication**
	- Rules based on packet headers. Many of these can be spoofed, no concept of user identity or authentication mechanism at the packet level. 


### Stateful packet filters
- A stateless filter examines each packet in isolation.
	- Judge purely on headers, bad for connectionless protocols
	- For protocols like TCP, two-way traffic, need inbound on port 80. 
		- But how does firewall know which inbound traffic is a legit response to something you initiated?
	- Adding rules for specific IPs = time-consuming, so did stateful intead.
- **A stateful packet filter** maintains a **connection tracking table** - a record of every active connection passing throughg firewall
	- When packet arrives, check against table, see if it determine to legit connection, or is unsolicited.
	- When ACK response arrives, firewall checks table, sees connection, sees src IP in response is same as destination in connection table, and is allowed through
	- Other packets =  dropped. 


### Application-level gateways
- Packet filters, even stateful ones, only operate at layers 2 and 3
- No visibility inside packet payload, only see headers
- **Application gateway** considers application layer protocol in use
- Some like HTTP and SSH will be allowed, others will be blocked outright
- An application level gateway sees:
	- Connection arriving on port
	- Inspects payload
		- Sees valid HTTP request on port 80, or something more malicious
- Can perform **much more complex port control** than fixed rules
	- Protocols that dynamically negotitate ports during their session like FTP
	- Application gateway understands protocols well enough to dynamically permit only negotiated ports for specific sessions and close them when the connection ends
		- Not possible with static packet filter rules 
### Proxy servers
- A **proxy server** initiates a connection on our behalf
- Sits between host and server, maintains request/reponse cycles independently between the two
- Does not passively monitor traffic
	- Receives request from client, makes decisions about whether to forward it according to packet filtering rules and application-level gateway. 
- Then opens own independent connection to destination, forwards the packet, receives response, inspects, returns to client. 

#### What proxy servers permit
- **Malware scanning**
	- Proxy receives complete response from web server, before forward, can pass through AV or sandboxx
- **Content caching**
	- Frequently requested resources, store at proxy, perf benefit
- **Logging and audit**
	- Every access passes through proxy, auditable.
- Behaves like application gateway, receives full application-level content in isolation and can examine before making forwarding decisions in tandem with packet filter.
#### Proxy server issues
- **Large overhead per connection**
	- Maintain two sep connections for TCP, buffer reqs and responses, perform scanning etc, more expensive than packet filtering.
- **Configuration complexity**
	- Clients must be configured to route traffic through proxy; hard to scale. 
- **Seperate server per service**
	- HTTP proxy handles HTTP and HTTPs, but inspection rules are diff for diff protocols, usually deploy multiple.

## Network Address Translation
- IPv4 has 4.3 billion addresses.
- Private address ranges define as non-routable, reused across millions of networks
- Translates local private addresses to public IPs and allows many machines on a network to share a single public IP.
	- When an outbound packet is received by gateway, rewrite private source IP to public IP, so responses sent to correct desination
	- Mapping stored in NAT table
	- NAT routers use different ports to track different connections, which is how multiple internal machines using the same public IP are distinguished, can then map back to correct private IP when recevigin inbound packet. 
- Result of this is that internal machines are almost totally hidden as a security consequence:
	- Can't send to private IP publicly
		- Not routable on internet, so packet dropped
	- Sends to public IP
		- Reaches NAT router, with specific port chosen by attacker
		- If no entry for chosen port for that mapping, then no internal machine initiated a connection, so dropped
- Cannot enumerate internal machines or confirm they exist
- So prevent unsoliticted attacks on random ports, but no defense against internal attacks, outbound initiated threats. 