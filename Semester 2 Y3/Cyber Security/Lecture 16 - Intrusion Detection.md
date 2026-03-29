## Why Intrusion Detection Exists
- Attacks using valid protocols e.g., SQL injection over port 80 HTTP from the internet, thus slips through the firewall
- Insider attacks never cross a firewall to begin with, and often have no visibility of internal behaviour unless have local firewalls, which can be bypassed depending on the level of internal threat, so are simply not enough.
- Thus, intrusion detection, as opposed to blockage, exists, which examines what is happening, rather than simply yielding a binary outcome as to whether traffic is permitted or not.
## Anti-virus
- These are computer programs that are used to prevent, detect, and remove malicious software from a subject's machine
- They encapsulate prevention of malware, detection of malware, and recovery/disposal from/of malware.
- There are different types of anti-virus each with different capabilities regarding malware detection:
	- **Signature-based detection**
	- **Heuristics**
	- **Machine learning/"Next gen"**

### Signature Detection
- Oldest and most straightforward approach to detecting malware.
- Security researchers obtain samples of malware and identify a sequence of bytes that is unique to that malware, that is:
	- It is present in every variant,
	- Not present in legitimate software,
	- Short enough to be computationally practical to scan for
- This usually comes from the core code which perfoms the malicious routine e.g., the encryption routine of ransomware, command and control server addresses, ransom note text, registry keys it creates.
- Anti-virus software scans files either in bulk, or at runtime of those files, and compares with the stored signature database it maintains of unique byte sequences
	- Note that specific byte sequences may not be contiguous; a signature can be composed of multiple fragments at specific offsets within a file.
		![](Pasted%20image%2020260328220438.png)
		 ![](Pasted%20image%2020260328220213.png)
- Malware authors quickly learned to evade specific signatures by making minor modifications; **polymorphic malware** mutates its own code between infections whilst preserving its function, changing the byte representation to make traditional signature detection harder.
- **Generic signatures** address this by identifying patterns at a higher level of abstraction - they specify detection rules designed to identify entire families of malware.
	- Usually maintain some structural information/hints to look for e.g., offsets a byte sequence might appear at, gaps that may appear between byte sequences, characteristics of ransomware to beware of, looking for multiple unknown links/strings (could by CNC servers) etc.
- Fundamental weakness is that this only catches malware already know about. Works for known malware families and patterns, but fails against zero-day exploits and malware using them, and will be entirely undetected.
### Heuristic Detection
- Heuristics attempt to address the zero-day problem by analysing behaviour rather than identity
- The key technique is **sandboxing**:
	- The suspicious program is ran inside an isolated virtual machine
	- We observe what it does: if it tries to enumerate all running processes, open network connections to unknown IPs, disable the firewall, replicate itself to other drives etc it can be considered malicious
- This can theoretically detect previously unknown malware that behaves like known malware, even if the bytes are completely different
- The obvious limitation is the sandboxing itself - sophisticated malware can detect if it's running in a sandbox and behave innocuously until it's in a real environment, or even escape from the VM if it is not configured correctly permitting real damage to be caused.
	- Hard to configure correctly against either a zero-day exploit in the VM provider or a piece of completely new and unknown malware.

### Machine learning/"Next gen"
- Traditional AV is fundamentally file-centric and based on moment-in-time maliciousness (checks at access, and then trusted thereafter for the foreseeable)
- Modern malware evades traditional AV by avoiding malicious files entirely
	- Attacker delivers Word Document with a macro
	- Macro runs powershell
	- Powershell downloads script from internet
	- Scripts runs entirely in memory, it is never committed to disk from the process' memory space
	- Script uses built in windows tools to achieve malicious goals
- Malicious behaviour only emerging from the combination and sequencing of valid components that remain fileless is impossible for traditional AV to detect
	- Delayed execution(may even download payload and run it later locally), code injection into another binary whilst malware binary is entirely clean etc.
- "Next Gen" AV maintains a wider scope of attributes on which to judge malware
- NGAV  maintains a continuously updated record of what every process is doing and builds the traditional picture of what a process does over a "learning period" during initial deployment.
- When a sequence and combination across time appears malicious, deviating from the baseline, then an alert can be triggered
	![](Pasted%20image%2020260328222454.png)
	Does not just monitor isolated events
- Often correlate across "dimensions" e.g., file system network, process, time to determine confidence level and response


## Intrusion Detection/Prevention
- **Intrusion Detection Systems** (IDS)
	- Observes, analyses, and alerts
	- Passive and does not interfere with traffic
	- Generates logs and notifications for administrators
- **Intrustion Prevention Systems** (IPS)
	- Everything IDS does, but also actively blocks or modifies traffic in real-time.
- IPS sounds strictly better but an IPS that makes incorrect decisions will block legitimate traffic, potentially disrupting operations
	- Often have human oversight.

### IDS Deployment
- **Host-Based**
	- Runs as software on an individual machine, monitoring that node's behaviour from the inside
	- Creates a profile of usage for specific users, creating a baseline, and determines deviation across multiple dimensions of resources in real-time to identify threats.
	- Can see:
		- CPU + memory usage patterns
		- Which applications are running
		- File system usage
		- Network activity coming to/from this host
		- System calls made by processes
		- User login activity
		- Log analysis
		- Registry changes (windows)
	- Big advantage is that it can see encrypted traffic after decryption since it runs on the endpoint where decryption happens (as opposed to NIDS)
	- Drawback is resource consumption, and the fact that a sufficiently sophisticated attacker who has compromised the host could potentially tamper with or disable the HIDS
- **Network-Based**
	- Placed at a strategic **viewpoint** on the network and analyses traffic flowing past it
	- E.g., installed on a firewall or in a DMZ or behind a screened subnet to catch insider threats.
	- Place at multiple points to give coverage at different layers against different attack vectors.
	- Can perform analysis beyond what a firewall does, such as stateful protocol analysis over-time and deep packet inspection
	- A firewall might just check port numbers whereas a NIDS can actually parse the contents of packets and look for anomalies within them, more on this below.

### Logical Components of IDS
- **Sensors/Agents**
	- Responsible for collecting or collating data from multiple viewpoints on a network
	- E.g., strategic point at network, host computer
	- This data is usually forwarded to an analyser (may do some lightweight processing onboard e.g., remove duplicate packets, drop obviously irrelevant traffic via filtering, but not things like deep packet inspection)
- **Analysers**
	- Responsible for ascertaining whether an intrusion has taken place
	- May be local on the agent(resource-intensive), or may be a centralised server receiving data from all sensors
		- Typically centralised though.
	- Performs the computational work e.g., signature matching, anomaly scoring, deep packet inspection
- **Reporters**
	- Results from analysis are forwarded to reporters; security operations console showing alerts from all sensors
	- Permits a correlated graphical view across the whole network as determined by the analyser.

### IDS Detection modes
- **Signature-Based**
	- Fingerprinting sequences of operations or packets
- **Anomaly-Based**
	- Build a baseline and find deviations much like unsupervised machine learning AV.
- **Stateful Protocol Analysis**
	- More complex version of a stateful packet filter.

#### Signature-Based Detection
- Signatures are created and stored in a database
- But network attacks are rarely single packets or at a precise moment-in-time
- A signature must therefore express more than just bytes, and one typically uses a rule language to specify, at a high level, what to prevent.
- These signatures and then stored, and network activity is compared against them.
	- Usually identify sequences of packets as a state machine, and determine whether currently in a state where the packet is part of a known attack sequence as according to the stored signatures.
 - Computationally efficient
- Always spots known attacks, but misses unknown attacks.
- For example:
	- What are the signs that a host on the network is performing host scans?
		- Large amounts of ICMP traffic
		- Many SYN packets
		- These connections going to a variety of other hosts
		- Can be interpreted as a rule "If a host establishes more than 3 TCP connections in 5 seconds, it's port scanning"

##### Snort
- Widely used open-source IDS that implements signature-based detection.
- The rule syntax is used to implement this detection mechanism:
	![](Pasted%20image%2020260329002430.png)


##### Signature Evasion
 - Signature evasion is often straightforward; they directly encode assumptions about attacker behaviour, allowing a sophisticated attacker to simply behave differently.
 - For example, Nmap's port scan detection signature relies on the rate of it's connection, and these are built into Snort.
 - Nmap provides 6 timing options:
	 ![](Pasted%20image%2020260329002658.png)
- By reducing the speed of the scan, the packets are indistinguisable from normal background noise.
- The time taken to complete the scan increases, but it will never trigger a signature that looks for multiple connections in a short window.
	- This highlights the weakness of signature-based IDS, you cannot possibly enumerate all the ways that attackers will behave.

#### Anomaly Detection
- Rather than defining concretely what is "bad" analysers build a statistical model of normal usage.
- When a deviation occurs from this baseline, it is flagged.
- The key advantage is that anomaly detection in principle can detect completely novel attacks.
	- If attacker does something never been done before, it will still deviate from the normal profile and raise an alert.
- Always a trade-off between false positives and false-negatives
	- If the detection threshold is too sensitive, it flags lots of normal behaviour, overwhelming administrators, and losing real alerts in the noise.
	- If it is too relaxed, there are fewer false positives, but more real attacks miss.
	- Need different thresholds for different security level and degree of manpower
- Defining normal:
	- Run the host/network in quarantined environment, monitoring audit logs, system calls, network behaviours
	- Collect data over a representative period to build a statistical model of normal behaviour
	- Then monitor live behaviour and compare against the model, flagging significant deviations (unsupervised learning).
##### Complex Behaviours and Machine Learning
- May not yield a simple bell curve, but may yield a distribution that has multiple peaks, valleys, and changes over time. 
	![](Pasted%20image%2020260329003544.png)
- It is difficult to determine whether something like this is anomalous, and simple statistical models will struggle with this.
- These non-linear relationships can be modelled using machine learning, making predictions on data.
- Support Vector Machines, Neural Networks etc all see use in intrusion detection
	- Take as input many parameters e.g., syscalls, network bandwidth, OS system calls to model non-linear patterns
- Pre-train on labelled data, both normal and attack traffic, and then deploy to classify live traffic in real-time to detect intrusions.


DRAWBACKS:
- Slow, low-volume attacks may never deviate enough
- Detection needs to be fast enough to be useful, with complex ML inference taking time, there is clearly a trade-off between model sophistication and speed.
- The more metrics used, the exponentially larger search space.
- Retraining problem; model may become stale, increase false positives. 


### Stateful Protocol Analysis
- Most sophisticated detection mode.
- Rather than looking for known signatures or deviations from statistical norms, it understands the protocols in use and flags semantically suspicious behaviour
- Requires the IDS to have a complete, accurate model of every protocol it monitors including:
	- What commands/messages are valid?
	- What are typical parameter sizes?
	- What sequences of operations are normal?
	- What legitimate state look like at each stage?
- Detects attacks that are novel(no signature) and statistically normal(no anomaly) but not within the allowed bounds of that specific protocol.


IN PRACTICE, USE ALL 3.