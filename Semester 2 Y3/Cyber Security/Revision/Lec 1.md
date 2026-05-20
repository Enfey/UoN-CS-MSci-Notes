## What do we mean by security
- **Defining security**
	- **<mark style="background: #FFF3A3A6;">Unbreakable vs secure enough</mark>**
	- <mark style="background: #FFF3A3A6;">Tradeoffs, risk assessment, cost/benefit</mark>
	- <mark style="background: #FFF3A3A6;">Arms race </mark>between researchers/malicious actors
		- <mark style="background: #FFF3A3A6;">Proactive</mark>(against assumed threats)
		- <mark style="background: #FFF3A3A6;">Reactive</mark>(against worst known threats)
- Security implemented via, three broad measures
	1. <mark style="background: #FFF3A3A6;">**Prevention**</mark>
		access controls, encryption
	2. <mark style="background: #FFF3A3A6;">**Detection**</mark>
		audit logs, intrusion detection
	3. <mark style="background: #FFF3A3A6;">**Recovery**</mark>
			backups, RAID, incident response plans
- **<mark style="background: #FFF3A3A6;">Perfect security</mark>** = not exist, unbreakable = <mark style="background: #FFF3A3A6;">unrealistic,</mark> design to be <mark style="background: #FFF3A3A6;">**secure enough**</mark> against known and predicted threats


## Managing security
- **Management responsibility** in organisation
	- They define, what must be protected, devs implement requirements
- They define, **security policy** detailing the requirements containing:
	- <mark style="background: #FFF3A3A6;">Assets</mark>
	- <mark style="background: #FFF3A3A6;">Threats</mark>
	- <mark style="background: #FFF3A3A6;">Acceptable use</mark>
	- <mark style="background: #FFF3A3A6;">Legal and regulatory constraints</mark>
- Govern password, rules, audit trails etc
- <mark style="background: #FFF3A3A6;">Security issues usually policy failures</mark>, not technical ones. 


### CIA triad
- Security objectives
- **Confidentiality**
	- Prevention, unauthorised disclosure of informatiom
		- Do not want to compromise secret/private info.
- **Integrity**
	- Prevention, unauthorised modification of information
		- Bank balances, database records, config files, not just about attackers, detect accidental corruption. 
- **Availability**
	- Prevention, unauthorised withholding of information/resources
		- DoS, ransomware, depends on redundancy, fault tolerance, backups; need to access on demand
		- Availability issues, often caused, non malicious issues e.g., hardware faults

## Accountability
- Users, should be, <mark style="background: #FFF3A3A6;">responsible, their actions</mark>
- System, identify, authenticate users, ensure compliance, keep audit trails
- Has conflicting values w/ privacy e.g., <mark style="background: #FFF3A3A6;">significasnt logging for compliance</mark>




### Non-Repudiation
- <mark style="background: #FFF3A3A6;">Ensures that someone cannot deny an action later;</mark> indisputable evidence they committed it
	- <mark style="background: #FFF3A3A6;">Digital signatures, certifficates</mark>, cryptocurrencies, keycards.
- <mark style="background: #FFF3A3A6;">Mostly, legal concept (because forgeries); </mark>evidence **verifiable by trusted 3rd party**

### Security vs Usability
- <mark style="background: #FFF3A3A6;">Security unaware users</mark>, have specific securty requirements, but <mark style="background: #FFF3A3A6;">no security expertise</mark>
- Security design, often results, in<mark style="background: #FFF3A3A6;"> **maximisation**</mark>
- Trade off <mark style="background: #FFF3A3A6;">between security + ease of use,</mark> strong security increaes friction, users have to work around controls
	- When those speceifiying security policy are inexperienced, usability suffers
	- E.g., excessive CAPTCHA, timeouts and lockouts, MFA
- GOOD SECURITY DESIGN IS ABOUT COMPROMISE, NOT MAXIMISATION.


### Data vs Information
- Security mechanisms, protect data, not information.
	- **Data** = raw bits
	- **Information** = meaning derived from the data
- Leads to <mark style="background: #FFF3A3A6;">**inference attacks**,</mark> where protected info can be revealed indirectly, even though data, is secure
- E.g., database search, joe's criminal record, u do not have permission, reveals it exists (may supply additional attack vectors).

### Tradeoffs
- <mark style="background: #FFF3A3A6;">Security vs usability</mark> (MFA, disk encryption, lockouts, timeouts, E2E, data collection vs personalisation)
- Accountabiility vs privacy (audit trails + logging)
- <mark style="background: #FFF3A3A6;">Overhead and costs </mark>(computation, communication, storage) and societal cost too.


## Security Design
### Security Focus
- Design in, from start, adding later, = incomplete, must be a systematic, disciplined approach.

### Security design
- Good security design f<mark style="background: #FFF3A3A6;">ocus on these principles</mark>
	- **<mark style="background: #FFF3A3A6;">Focus of control</mark>**
		- Should focus of control be data? operations? users?
		- Mix all 3 e.g., file perms for users, API restrictions, database constraints for data.
	- **<mark style="background: #FFF3A3A6;">Complexity vs assurance</mark>**
		- Simple systems easier to audit and verify, feature-rich systems harder to secure and reason about
	- **<mark style="background: #FFF3A3A6;">Centralised or decentralised controls</mark>**
		- We ask, should defining + enforcing sec, be performed, single entity, or left to individual components in a system 
			- Single entity = bottleneck
			- Decentralised = scales better, harder to manage/reason about. 
	- **Layered security**
		![](Pasted%20image%2020260204175924.png)
		- Can visualise security through lense of abstraction
		- Each layer relies on security of layers below it. 
		- ![](Pasted%20image%2020260204180006.png)
		- if cannot break at one layer, just attack layer underneath
		- DATABASE QUERY, APPLICATIONS, SERVICES (SQL ENGINE, TRIGGERS FILE WRITE, INCORRECT PERMS, BUG, HDD WRITE, SECURITY IN ANOTHER LAYER FAILED, COMPROMISING SYSTEM)