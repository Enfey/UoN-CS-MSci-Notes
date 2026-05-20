# Windows architecture
- Windows = layered architecture; separates architectural components based on operating modes
	- **User mode**
		- Isolated from direct hardware access, go through system call interface.
			- User processes, system processes, service processes, security subsystem, syscall interface. 
	- **Kernel mode**
		- Kernel, drivers, memory management, reference monitor implementation enforcing OMAC
	- **Hyper-V**
		- run beneath OS in virtualised environments.

### Security subsystem
- Runs in **user mode**; central to auth
- Components
	- **Logon processes (Winlogon)**
		- Handle secure logon interaction; display secure logon screen, pass to LSA
	- **Local Security Authority**(LSA)
		- Validate user credentials(pass hash to SAM), create access tokens, enforce local security policy.
	- **Security Account Manager**(SAM)
		- Stores local user account database with hashed passwords used for local auth. 

### Access control matrix
- Access rights are defined individually for every combination of subject and object.
	- Rows  =  subjects, columns = objects
	![](Pasted%20image%2020260219002238.png)
- Very fine grained; conceptual. Would be impractical to store and maintain. 

### Row-Based Implementation
- For a given subject, hold a list of its permitted actions on all objects. Not used on windows.

### Column-Based implementation - Access Control Lists (ACLs)
- For a given **securable object**, denote which trustees have what rights regarding the objects. 
- This is the form taken on windows, on a per-securable object basis.
- Extend RWX with:
	- Delete, change permissions, take ownership
- Specifies the access rights for that trustee (allowed, denied, logged).


### Access control
- Access is granted based on the principal's access token, and the objects **security descriptor**
- A **securable object** is an object that can have ownership, permissions, and auditing. 
- A **security descriptor** is held by a securable object:
	- **Owner SID**
	- **Primary group**
	- **DACL(Discretionary)**
		- Controlled by object owner
		- Access control, can't log
	- **SACL(System)**
		- Managed by admins, defines which actions are logged/audited.
		- Logging, can't block.
	- These ACLs are maintained separately.
- ACLs in windows have inheritance e.g., securable objects in dir inherent from parent directory



### Principal 
- A principal is an entity that can be assigned permissions:
	- Local users, domain users, groups, machines
- Each principal has:
	- Human readable name
	- Security identifier (**SID**)
		- When principal authenticates, their access token includes their SID, and SIDs of groups they belong to. 

### Local/Domain principals
- LSA creates local principals - users, groups, servers, authenticated on standalone computer
	- **username = MACHINE\username**
- A **windows domain** is a centrally managed network of computers and users that share a common auth system
- Domain princpals are managed centrally by domain admins, via a **domain controller**
	- username@domain = DOMAIN\principal
	- Identifies the user and domain that auths them, tells windows to not auth locally, defer to domain controller
	- Can use net commands to query domain information from a domain-joined machine such as retrieving local groups on the machine, showing the domain members inside them


### Groups
-  A group is a collection of SIDs
- A group is also an SID
- Groups can thus be nested, though this is not the case on local machines; it is only possible via **Active Directory**
- All domain groups are stored in **Active Directory**
	Then replicated across domain controllers to be managed centrally

### Objects
- Passive entities in access operations
	- Threads, files, directories, registry keys
- On windows if it has **permissions**, **ownership**, and **auditing**, then it is a **securable object.**
- Securable objects:
	- Owner SID
	- DACL
	- SACL
	- Primary group


### Subjects
- Those entities which are active and attempt to access objects.
- Windows subjects; processes, threads.
- Every subject carries an access token; when a process creates a new process, it gets a copy of the parent access token, possibly slightly modified. 
	- When login, LSA creates access token, then edge starts with that token, then chrome is invoked via edge etc. 
	- Modified e.g., run as admin, get admin access token, denotes access capability.


### Access tokens
- An access token = data structure storing security credentials for a login session
- Identifies **user, user groups, user privileges.**
	- **User + Group SID**
	- **Privileges**
	- **Defaults for new objects**
	- **Misc.**
- When subject tries to access object:
	- Compare subject access token against objects security descriptor (DACL) then grant or deny access
-  Tokens are copied and not shared as they are immutable.
	- Prevents one process modifiying another's security context.
	- But have TOCTTOU, already running processes have same privileges, remain until process restart. 

### User Account control 
- After vista, token model changed; users have two tokens:
	- Restricted token - used by default for most actions
	- Full token - used when elevated privileges are required.


### Domains
- Windows domain is a centrally managed network of computers that share a common auth system
	- SSO, centralised admin etc
- Centralised security
- A domain controller is responsible for managing authentication requests:
	- This is a trusted 3rd party responsible for enforcing security policies and authentiation centrally
	- Stores user accounts and details and ACLs in a central database called **Active Directory**
		- This is the identity management system specifically for windows domains
		- Auth checks against this. 
		- We say ACLs but ACLs here only pertain to domain specific objects; e.g., who can reset password. ACLs are per-system
- Multiple DCs allow for decentralisation/fault tolerance/uptime/ just replicate/shard.
- Instead of each PC then managing its own accounts, user accounts stored in AD, assign domain groups, and propagate DACLs and SACLs to networked PCs.


### Interactive logon
- Where user logs into machine locally; triggered by control alt delete; called the **SECURE ATTENTION SEQUENCE**
	- No normal program can intercept this key sequence, guarantee Winlogon responding, without it, could spoof logon screen. 
- **LOCAL LOGON**
	- SAS
	- Graphical identification and authentication (GINA) displays logon screen.
	- User enters credentials. 
	- GINA pass to LSA
	- LSA uses NTLM
		- Legacy auth protocol - challenge response to auth user against SAM
	- Successful login provides subject with access token that then dissemninates throughout the system; spawns the user shell.
- **REMOTE/DOMAIN LOGON**
	- Replaces NTLM with kerberos, a ticket-based authentication protocol with verification.
	- Replaces SAM with a domain controller, and by extension, Active Directory to authenticate that domain principal. 

