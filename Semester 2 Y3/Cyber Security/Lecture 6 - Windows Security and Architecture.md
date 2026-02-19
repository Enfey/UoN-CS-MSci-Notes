# Windows architecture
- We can denote windows as a layered architecture, separating architectural components based on operating modes.
	- User mode - isolated from direct hardware access, must go through system call interface to interact with the kernel. 
		- User processes
		- System processes
		- Service processes
		- Environment subsystems
		- Security subsystem
		- System call interface
	- Kernel mode
		- Kernel
		- Kernel-mode drivers
		- Memory manager
		- Reference monitor implementation enforcing access control decisions, 
	- Hyper-V
		- Runs benath the OS in virtualised environments. 

## Security Subsystem
- Runs in **user mode**, but is central to authentication
- Components:
	- **Logon processes** (Winlogon/LogonUI) 
		- Handles secure logon interaction with independent users, displaying the secure logon screen, and collects credentials and passes them to LSA. 
	- **Local security authority (LSA)**
		- Validates user credentials
		- Crates access tokens
		- Enforces local security policy
		- Checks against SAM.
	- **Security account manager (SAM)**
		- Stores local user account database
		- Hashes passwords
		- Used for local authentication

## Windows Access Control Model
- Windows predominantly uses **Access Control Lists**; a security mechanism maintaing a list of access control entries, where each entry identifies a trustee and specifies the access rights allowed, denied, or audited for that trustee.
- Extends the usual RWX with fine-grained permissions such as 
	- Delete
	- Change permissions
	- Take ownership
- This gives more flexibility
- But subsequently, also yields more complexity and difficulty to manage semantically.

## Access control matrix
- Access rights are defined individually for each combination of subject and object
	![](Pasted%20image%2020260219002238.png)
- This of course, is quite conceptual, but would allow for very fine grained control. 
	- This would be impractical to store due to size on complex systems, even with a good degree of optimisation. 

### Capabilities
- Row based implementation of the access control matrix, where a list of capabilities is defined per user, globally.
	![](Pasted%20image%2020260219002853.png)
- Each subject then, holds a list of its permitted actions on objects
- This is rarely used in windows. 

### Access control list
- Stored with an object/relating to a program
- Corresponds directly to a column of the access control matrix; denoting which principals have what rights regarding the object. 
- Cheaper than ACM, don't store those permissions which would otherwise be empty. 
- This is the form that Access Control Lists(ACLs) take on windows; implemented on a per-securable-object basis that is.
	![](Pasted%20image%2020260219002845.png)

## Access control
- Access is granted based on the **principal**'s access token and the object's security descriptor
- Access control in windows treats more than just files, if an object can have **ownership**, **permissions**, and **auditing**, then is is a **securable object**
	- Each **securable object** has a **security descriptor** containing:
		- Owner SID
		- Primary group
		- DACL(Discretionary)
			- Controlled by object owner
			- Allows/denies entry.
		- SACL(System)
			- Managed by administrators to log, or audit, attempts to access secured objects. 
			- Logs/auditing. 
- Inheritance is implemented regarding ACLs; objects can inherit ACLs from parents e.g., dir inherits from parent directory. 


### Principal
- A principal is an entity that can be assigned permissions:
	- Local users
	- Domain users
	- Groups
	- Machines
- Each principal has: 
	- Human-readable name
	- Security Identifier (SID)
		- When a principal authenticates, their access token includes their SID, and all group SIDs they belong to.

#### Local/Domain Principals
- LSA creates local principals - users, groups or servers, that are authenticated on a standalone computer or member server - on the individual machine 
	- username = **MACHINE\username**
- A **windows domain** is a centrally managed network of computers and users that share a common authentication system, SSO capabilities, centralised administration etc
- Domain principals are managed centrally by domain admins on a domain controller within an enterprise environment
	- username@domain = DOMAIN\principal
	- Identifies the user and the domain that authenticates them, telling windows to not check authentication locally, but instead against the centralised domain that grants that user access. 
	- Can use `net` commands to query domain information froma  domain-joined machine
		- `net user /domain` (retrieves domain user accounts)
		- `net group /domain` (retrieves domain groups)
		- `net localgroup /domain` (retrieves local groups on the machine, showing the domain members inside them)


### Groups
- A group is a collection of SIDs
- Group itself can be an SID
- Groups can thus be nested, though this is not the case on local machines, not possible in **SAM**, only possible in **Active Directory** which supports hierarchical group structures. 
- All domain groups are stored in Active Directory and are replicated across Domain Controllers to be managed centrally. 

### Objects
- These are passive entities in access operations
	- They are being accessed e.g., threads, files, directories, registry keys, processes, etc. 
- If an object can have **ownership**, **permissions**, and **auditing**, then is is a **securable object**
- Securable objects have:
	-  **Owner SID**
	- **Primary group**
	- **DACL(Discretionary)**
		- Controlled by object owner
		- Allows/denies entry.
	- **SACL(System)**
		-  Managed by administrators to log, or audit, attempts to access secured objects. 
		- Logs/auditing. 


### Subjects
- Subjects are those entities which are active; they attempt to access objects.
- Windows subjects are typically processes and threads.
- Every subject carries an **access token**; when a process creates a new process, it gets a copy of the parent access token, possibly slightly modified
	- When you log in for example, LSA creates your access token, then edge starts with that token, then chrome is invoked via edge, and then chrome gets a copy of the access token. 
	- Modified e.g., run as administrator. 
### Access tokens
- An access token is a data structure storing security credentials for a login session
- Identifies the user, the user's groups, and the user's privileges
	- **User SID**
	- **Group SID**
	- **Privileges**
	- **Defaults for New Objects**
	- **Miscellaneous**
- When a subject tries to access something, the kernel compare's the subject's access token vs the object's security descriptor (DACL), then grants or denies access. 
- Tokens are copied and not shared, because tokens are immutable, and this prevents one process from modifying another' security context. 
	- But because of this, create TOCTTOU issue, the already running processes would still have the same privileges, remain until logoff/process restart. 

#### User Account control
- After Vista, admin users don't use an admin accss token by default
- Users have **two** access tokens
	- Restricted - used by default for most actions
	- Administrative - used when elevated privileges are required. 

## Domains
- A **windows domain** is a centrally managed network of computers and users that share a common authentication system, SSO capabilities, centralised administration etc
- Centralised security
- Domain controller:
	- Trusted 3rd party responsible for user authentication and enforcing security policies centrally
	- Stores user accounts and details and access control in a central database (Active Directory)
		- Identity management system specifically for Windows domains.
		- What authentication checks against for domain users. 
		- We say access control, but ACLs are really only here for domain specific objects e.g., Active Directory Objects, so define who/what groups can reset a password. 
- Multiple DCs allow for decentralisation/fault tolerance/uptime etc by design; just replicate the directory/logic. 
- Instead of each PC then managing its own accounts, user accounts are stored in AD, and permissions are often assigned via domain groups.


## Interactive logon
- This is where a user logs into a machine locally
- It is triggered with `CTRL + ALT + DELETE`, called the **Secure Attention Sequence**
	- No normal program can intercept this key sequence, guarantees that Winlogon is responding, without it, could spoof/display fake login screen. 
- The **local logon** process contains (Pre-Vista)
	- `CTRL+ALT+DELETE`
	- Graphical Identification and Authentication (GINA) displays login screen
	- User enters credentials
	- GINA sends credentials to LSA
	- LSA uses NTLM
		- NTLM is a legacy authentication protocol (challenge response), hashes the password, verifier sends challenge, client encrypts using password hash, verifier verifies response. 
	- NTLM checks against SAM database
	- Successful login provides an access token which is used to spawn the user's shell (explorer.exe), allowing access to the system
- The **domain logon** process:
	- Replaces NTLM with kerberos
		- Modern ticket-based authentication protocol, instead of sending password-based responses, use a trusted third party
		- User requests auth from DC, DC verifies credentials, and issues a **ticket-granting ticket**, proving the user has been authenticated. 
		- When accessing resource, client sends TGT to DC, requests service ticket for specific server, DC issues **service ticket**
		- Client then presents service ticket, and server verifies that, and access is granted. 
		- Tickets expire, and scales well, mutual authentication of client and server (when present service ticket, must be valid).
	- Replaces SAm with a domain controller, and by extension, an Active Directory within that domain controller to authenticate the domain principal.

### Interatctive logon
- The windows interactive logn