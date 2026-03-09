## Firewalls
- A firewall is a hardware/software system that sits at network boundaries.
- Controls which packets are allowed to pass between them
- *All traffic leaving any subnet must pass through the firewall* - a firewall you can route around provides no protection.
	![](Pasted%20image%2020260308182753.png)
### Firewall Functions
- **Single point security** - Enforces policy at network boundary - everything inside is implicitly trusted, and everything outside is implicitly untrusted.
- **Packet analysis and logging** - every packet passing through can be inspected and logged, providing security event record and visibility into network activity. 
- **RBAC** - defined ruleset determining what is allowed and what is blocked, defining network behaviour via policy

### Firewall Location
- **Network Firewalls**
	- Placed between a subnet and the internet - protects an entire network with a single device. 
	- Home router is the most familiar example, sits between home network and ISP, blocks all unsolicited inbound connections by default and allows all outbound traffic. 
		- Enterprise network firewalls do the same at much larger scale with far more sophisticated rulesets. 
	![](Pasted%20image%2020260308182956.png)
- **Host-based Firewalls**
	- Placed on individual machines and filter traffic at the machine level rather than the network level.
	- Defends against compromised machines on internal network and attackers who bypassed the network firewall, rogue employees.
	- Never assume anything on internal network is safe. 

### Demilitarised Zone
- Architectural concept, small subnet that separates externally facing services from the internal network.
- It sits between the internet and the internal network, separated by network-level firewalls on both sides. 
	![](Pasted%20image%2020260308184053.png)
- An attacker who compromises the web server is in the DMZ and has to bypass another firewall. 
- The inner firewall can be configured to allow very limited traffic from the DMZ to the internal network. 


### Firewall Basic Functions
- Defending against external access
	- Prevent internet parties from accessing internal services, without a firewall, every service running on every internal machine is potentially reachable from anywhere on the internet.
- Restricting internal to external access
	- Firewalls can control what internal users are allowed to connect to externally. 
	- E.g., IRC, historically used as command and control channel for botnets, a compromised machine would connect outbound to an IRC server and receive instructions, since IRC is outbound traffic, a firewall only filtering inbound traffic wouldn't stop it. 
- Network Address Translation
	- See further down. 

### Firewalls are not sufficient on their own
- **Tunnelling**
	- If an attacker can encapsulate malicious traffic inside a permitted protocol, the firewall cannot distinguish.
	- HTTPS on port 443 is almost universally permitted - malware communicating over HTTPS looks identical to legitimate web traffic at the packet level. 
		- A firewall permitting HTTP/S permits anything that can be tunnelled over HTTP/S
	- DNS tunnelling encodes data in DNS queries which firewalls rarely block. 
- **Internal threats/insiders**
	- Only protect permiter; controlling outbound traffic and host firewalls can help, but if an attacker has physical access or access to multiple machines, this mitigation does not achieve much. 
- **Virus-infected files**
	- A firewall sees a permitted HTTP connection downloading a file, outbound. It does not inspect the returned packets for malware, the packet filter has no idea whether the bytes inside constitute legitimate software. 

## Packet Filters
- Packet filters are the most fundamental firewall mechanism
- They examine individual packets and apply rules based on **header information only**: 
	- Source IP
	- Destination IP
	- Protocol (transport layer)
	- Source port
	- Destination port
	- etc
- Possible for both inbound and outbound traffic.
- Operates at layers 2 and 3, with no knowledge of what application generates the traffic or what the data contains. 

### Packet Filter Rules
- Rule execution depends on implementation
	- IPTABLES: First rule to match is applied, rules evaluated in order. 
	- PF - all rules are evaluated, the last matching rule wins. 
- Every packet filter has a default action for packets that match no rule, which is the most important policy decision. 
	- Usually default ACCEPT or default DROP. 
	- Easy to accidentally DoS yourself by blocking necessary traffic.

### IPTables
- Userspace tool for configuring Linux's `netfilter` kernel firewall. 
- IPTABLES is just the configuration, `netfilter`'s modules constitute the actual packet processing framework running in the kernel. 
#### Tables and Chains
- IPTABLES organises rules into **four default tables** serving different purposes
	1. **Filter** - *the default table,* used for allowing and blocking packets, contains INPUT, OUTPUT, and FORWARD chains. 
	2. **NAT** - handles Network Address Translation, rewriting source and destination addresses.
	3. **Mangle** - packet alteration, used for TTL modification etc
	4. **Raw** - skips connection tracking; used for performance optimisation. 
- Within each table, **rules are organised into chains,** which have ordering:
	- **INPUT chain** - packets destined for this machine/gateway specifically.
	- **OUTPUT chain** - packets generated from the local machine/gateway.
	- **FORWARD chain** - all other packets passing through.
- Matches result in a jump, else we check the next rule.
	- A jump can go to ACCEPT, DROP, LOG, or another chain (chains can be added).
		![](Pasted%20image%2020260308193048.png)
- Apply the first rule that matches. 

#### Rules example
- Using the command line, we add rules onto the end of chains`
	![](Pasted%20image%2020260308193714.png)
- Notice the OUTPUT rule with `--sport80` allows any TCP packet leaving with source port 80 regardless of packet contents. It is stateless.
#### Types of Policies
- **Permissive**
	- Allow everything except for dangerous services. 
	- It is fundamentally reactive, you can only block things you know about. 
	- Unknown threats, forgotten services, and newly installed software are all automatically permitted. 
	- The default policy then, is ACCEPT.
- **Restrictive**
	- Block everything except designated useful services. 
	- More secure by default, but fairly easy to DOS onesself. 
		- If you set default DROP and forget to allow DNS requests, machine(s) will be unable to resolve names. 
		- Each omission silently breaks something, and debugging a restrictive firewall can be difficult, very delicate.

### Packet Filter Issues
- Simple, low level, and have high assurance. 
- But:
	- **They cannot prevent against attacks that employ application-specific vulnerabilities.** 
		- No idea whether packet payload is a legitimate request or contains malicious content. 
	- **Easy to accidentally allow or deny packets incorrectly**
		- Rule ordering is critical in first match configurations like IPTABLES.
		- A single misplaced rule can either facilitate attacks or break legitimate traffic.
		- This is especially the case for more complex rulesets.
	- **No higher-level authentication**
		- Rules are based on IPs, port numbers etc. Many of these can be spoofed, there is no concept of user identity, certificates, or any authentication mechanism at the packet level. 
### Stateful Packet Filters
- A stateless filter examines each packet in complete isolation - it has no memory of any previous packet. 
	- Each packet is judged purely on its own headers.
	- Creates a problem with two-way protocols like TCP; for a response to reach you, inbound traffic must be permitted. 
		- But how does the firewall know which inbound traffic is a legitimate response to something you initiated vs a unsolicited response?
	- Adding rules for every specific IP is time-consuming, so stateful packet filters were introduced. 
- A **stateful packet filter** maintains a **connection tracking table** - a record of every active connection passing through the firewall. 
	- When a packet arrives, it is checked against this table to determine whether it belongs to an existing legitimate connection or is unsolicited. 
		![](Pasted%20image%2020260308200054.png)
	- When an ACK response packet arrives, the firewall checks the table, sees the matching established connection, and sees that the src IP in the response is the same as the destination in the connection table, and is allowed through.
	- Unsolicited packets arriving from random IPs will be dropped. 


### Application-level gateways
- Packet filters, even stateful ones, operate at layers 2 and 3. 
- They have no visibility inside the packet payload - they only see headers of these corresponding layers. 
- An application gateway considers the **application-layer** protocol that is in use, rather than just the transport layer. 
- Some protocols will be allowed like HTTP and SSH, and others may be blocked outright.
- An application-level gateway sees:
	- Connection arriving on port
	- Inspects payload
		- Sees valid HTTP request for port 80
		- OR sees BitTorrent handshake disguised as HTTP.
- Can perform much more complex port control than fixed rules.
	- Protocols that dynamically negotiate ports during their session like FTP.
	- Application gateway understands protocol well enough to dynamically permit only negotiated ports for specific sessions and close them when call ends.
		- This is not possible with static packet filter rules, which would need permanent rules for all possible ports that a protocol can negotiate, facilitating massive attack surface. 

### Proxy Servers
- A proxy server initiates a connection on our behalf. 
- Sits in the middle between host and server, and maintains request-response cycles/connections independently between the two.
- The proxy does not passively monitor traffic, it receives the complete request from the client and makes its own decision about whether to forward it according to packet filtering rules and application-level gateway.
- It then opens its own independent connection to the destination, forwards the packet, receives the complete response, potentially inspects it, and returns it to the client. 
	![](Pasted%20image%2020260308201705.png)

#### What proxy servers permit
- Malware scanning
	- Proxy receives complete response from web server, before forwarding, can pass content through antivirus engine, check file hashes against known malware databases, or sandbox executable files.
- Content caching
	- Frequently requested resources can be stored at the proxy, significant performance benefit.
- Logging and audit
	- Every request passes through the proxy, providing a complete record of all web access. 
- Behaves like an application gateway as it receives the full application-level content in isolation and can examine it before making forwarding decisions in tandem with packet filtering rules on-proxy. 


#### Proxy Server issues
- Large overhead per connection
	- Must maintain two separate connections for example for TCP, buffer complete requests and responses, perform application-layer inspection, scan content etc. More expensive than just a packet filter, which can handle millions of packets per second.
- Configuration complexity
	- Clients must be configured to route traffic through the proxy. Hard to scale.
- Separate server per service usually
	- HTTP proxy handles HTTP and HTTPS, means deploying specialised proxies, or choosing protocols to benefit from proxying. 

### Network Address Translation
- IPv4 has approximately 4.3 billion addresses, with there existing many more devices than that.
- Private address ranges are defined as non-routable (`10.x.x.x, 172.16.x.x, 192.168.x.x`), and are reused across millions of networks worldwide. 
- Translates local private addresses to public IPs, and allows many machines on a network to share one public IP.
	- When an outbound packet is received by the gateway, the router rewrites the private source IP to a public IP, so that responses are sent to the correct destination.
	- This mapping is then stored in a NAT table, as a singular entry.
	- NAT routers use different ports to track different connections, which is how multiple internal machines using the same public IP are distinguished, and can map back to correct private IP when receiving inbound packet. 
		![](Pasted%20image%2020260308203800.png)
- The result of NAT is that internal machines are almost totally hidden, as an additional security consequence. 
	- An attacker attempts to reach an internal machine
		- Sends packet to private IP directly
			- Private IP not routable on public internet, so packet dropped somewhere on the internet.
		- Sends packet to public IP
			- Reaches NAT router, with specific port chosen by attacker
			- If there is no entry for the chosen port, then no internal machine initiated that connection, so it is dropped. 
	- Zero way to enumerate internal machines, address them, or get feedback they even exist. 
	- Thus NAT prevents unsolicited attacks on random ports, but no other types of attacks e.g., internal threats, outbound initiated threats e.g., visit malicious website, was initiated by machine and unless application-level gateway, ur FUCKED.