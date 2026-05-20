## Why intrusion detection exists
- Attacks using valid protocols that aren't sanitised via application gateways etc, slip through firewall
- Insider attacks usually never cross firewall to begin within, and have no visibility of internal behaviour without inside firewalls
- Thus, intrusion detection, opposed to blockage, examines what is happening, rather than just yielding binary outcome. 

## Anti-Virus
- Computer programs designed to prevent, detect, and remove malicious software from subject machine
- Encapsulate prevention, detection, and recovery/disposal from malware.
- Different types of anti virus:
	- **Signature-Based Detection**
	- **Heuristics**
	- **Machine-learning/next gen**

### Signature detection AV
- Most straightforward, acquire samples of malware and identify sequence of bytes unique to that malware that is:
	- Present in every variant
	- Not present in legitimate software
	- Short enough to be computationally feasible to scan for
- Usually comes from core code/routine which performs malicious routine e.g., CNC addresses, ransom note text, registry keys it creates
- AV scans files in bulk, or at runtime, compares with stored signature in DB
	- Specific byte sequences may not be contiguous; signature can be composed of multiple fragments at specific offsets within a file. 
		May also list file size, where may appear in file etc
- Malware authors learned to evade specific signatures by making minor modifications: **polymorphic malware** mutates its own code between infections whilst preserving its function changing the byte representation. 
- **Generic signatures**
	- Address this, identify pattern at higher level of abstraction
	- Usually maintain structural info/hints to look at, offsets a byte sequence may appear at, characteristics of ransomware to be aware of, gaps that may appear between byte sequences
- Only catches known malware families and patterns, fails against zero-day exploits

### Heuristic AV
- Address zero-day problem by analysing behaviour rather than identity
- Key technique is sandboxing:
	- Run sus program in VM
	- Observe; if it tries enumerating all running processes, open network connections to unknown IPs, disable firewall, replicate to other drivers, is malicious
- Can theoretically detect previously unknown malware
- Limitation = sandboxing
	- Sophisticated malware can detect if running in malware and behave like normal program until in real environment
	- Or even exploit the sandbox and escape entirely. 
	- Hard to configure correctly against zero-day exploit in VM provider 

### Machine Learning/"Next gen"
- Traditional AV = **file centric,** based on moment in time maliciousness
	- Checks at access, then trusted thereafter
- Modern malware evades traditional AV by avoiding malicious files entirely
	- Deliver Word Doc with macro that runs powershell and downloads script from internet
	- Runs in memory, never committed to disk, and uses official tools FILELESS
- Malicious behaviour only emerging from the combnination and sequencing of valid components that remain **fileless** is impossible for traditional AV to detect
	- Code injection into another binary whilst malware is entirely clean, delayed execution (download payload and defer execution) etc
- Next gen AV maintains a wider scope of attributes on which to judge malware. 
- NGAV maintains updated record of process actions, build traditional baseline picture of what apps do during initial deployment
- When sequence and combination across time appears malicious. deviating from baseline, alert can be triggered
- Often correlate across dimensions 


## Intrusion detection/Prevention
- **Intrusion Detection System(IDS)**
	- Observe, analyse, alert; gen logs and notifs for admins
	- Passive, does not interfere with traffic
- **Intrusion Prevention System(IPS)**
	- Everything IDS does, but also actively blocks or modifies traffic in real time
- IPS sounds strictly better, but IPS very easy to misconfigure, make incorrect decisions, cause DoS, need human oversight too. 


### IDS deployment
- **Host based**
	- Software on machine, monitor that node's behaviour
	- Create profile of usage for specific users, create baseline, determine deviation across multiple dimensions of resources
	- Sees:
		- CPU+memory usage
		- Which application are running
		- File system usage
		- Network activity
		- User login activity
		- Log analysis
		- Registry changes
		- Syscalls
	- Big advantage, can see encrypted traffic after decryption (as opposed to NIDS)
	- Drawback, resource consumption and sophisticated attacker who has compromised host could disable the HIDS
- **Network based**
	- Placed at **strategic viewpoint** on network and analyses traffic flowing past it
		- Not on device
	- Place at multiple points, give coverage at diff layers against diff attack vectors
	- Can perform analysis beyond what firewall does such as stateful protocol analysis **over-time** and deep packet inspection.
	- Firewall might just check port numbers and header, may be stateful packet filter, but NIDs can check payload contents, and doesn't necessarily need traffic to pass through it like an application gateway proxy server, and can analyse WHOLE packet. 

### Logical components of IDS
- **Sensors**
	- Responsible for collecting data from viewpoints on network
		- E.g., strategic point on network, host computer
	- Then forward to analyser (may do lightweight processing onboard e.g., remove dupe packets)
- **Analyser**
	- Responsible for ascertaining whether intrusion take place
	- May be local on agent (host-based) or may be centralised server receiving data from all sensors
		- Typically centralised
	- Performs computational work e.g., signature matching
- **Reporters**]
	- Results from analysis forwarded to reporters; security operation console showing alters from all sensors

### IDS detection modes
- **Signature-based**
	- Fingerprinting sequences of operations or packets
- **Anomaly-based**
	- Build a baseline and find deviations much like unsupervised learning next gen AV. 
- **Stateful protocol analysis**
	- More complex version of stateful packet filter

#### Signature-Based Detection
- Signatures created + stored in DB
- Network attacks rarely single packets or precise moment-in-time
- Signature must express more than bytes; one uses rule language to specify at high level what to prevent.
- Compare network activity against these stored signatures. 
	- Identify sequences of packets as a state machine, determine whether currently in state where this packet is part of known attack sequence according to sigs
- Computationally efficient
- Always spot known attacks, miss unknown
- E.g.,
	- What are signs that host on network is performing scans?
		- ICMP traffic, SYN packets, connections going to other hosts
		- If host establishes more than 3 TCP connections in 5 seconds, its port scanning
##### SNORT
- Open-source IDS, implements signature-based detection
- Rule syntax


##### Signature Evasion
- Straightforward; cannot possibly enumerate all ways attackers will behave, just behave differently
- E.g., nmap port scan, just change timing options, reduce speed of scan, so sig based IDS not triggered. 


### Anomaly based detection
- Rather than defining what is bad, build model of normal usage
- When deviation occurs, flag
- Detects completely novel attacks
	- IF attacker does something never done before will stll deviate from normal profile
- However, false-positives and negatives
	- Depends entirely on threshold
	- Need diff thresholds for different security levels. 
- Defining normal:
	- Run host in quarantined environment, monitor audit logs, syscalls, network behaviours
	- Monitor live behaviours, compare against model baseline, flag deviations

#### Complex Behaviours and machine learning
- May not yield bell curve via unsupervised, may yield distribution that has peaks, valleys, changes over time and per dimension
- Very difficult to determine whether something that looks similar to somthing in past is anomalous or not, entirely non-linear relationships
- Use machine learning, all see use in intrusion detection, take as input many parameters, pre-train on labelled data; real vs attack traffic, then deploy to classify live traffic. 


#### Drawbacks
- Slow, low-volume, methodical attacks may never deviate enough
- Detection needs to be fast enough to be useful, complex ML inference takes time, trade off between model capability and speed
	- Worse with more metrics, higher degree search space
- Retraining: baselines become stale, increase false positives

### Stateful protocol analysis
- Most sophisticated detection mode
- Understands protocols in use
- Require IDS to have complete, accurate model, of every protocol it monitors:
	- What commands/messages are valid? 
	- What are typical param sizes?
	- What sequence of operations is typical? 
	- What legit state looks like at each stage?
- Detects attacks that are novel(no sig) and statistically normal(no anomaly) but not within allowed bounds of that specific protocol

IN PRACTICE MIX ALL 3./ 