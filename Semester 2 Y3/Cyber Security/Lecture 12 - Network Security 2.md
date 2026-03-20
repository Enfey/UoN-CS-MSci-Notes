## Network Segmentation
- Practice of dividing computer networks, into smaller, isolated sub-networks
- Segmentation can be physical or logical:
	- **Virtual Local Area Networks (VLAN)**
		- Operates at Layer 2, the data-link layer, of the OSI model. 
		- A single physical switch can host multiple VLANs, with devices on different VLANs being unable to communicate directly - they must go through a router, and optionally a network-level firewall. 
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

## Zero Trust Architecture


## WiFi Security

