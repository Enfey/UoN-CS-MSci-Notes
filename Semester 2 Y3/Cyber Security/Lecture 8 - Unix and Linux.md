## Role of the OS
- An OS combines:
	- **Resource/device management** - CPU, memory, storage
	- **Identification + authentication** - verifying user identity
	- **Access control/authorisation** - permitting access to objects according to principal access rights/authorisation
	- **Auditing** - logging actions for bookkeeping/accountability
	- **User account management** - storing permissions
	- **Configuration**
- The OS's security components behave as a security kernel - an implementation of a reference monitor; an access control concept that refers to an abstract machine that mediates all access to objects by subjects/principals. 
- All accesses must pass through this security kernel concerning the guarded objects, and is simple enough to formally reason about.
## Authentication & Authorisation
- Assuming we've authenticated (proving identity via possession of some artefact only the identity claimant should/could be in possession of), the system must enforce **authorisation**.
- The core security model components:
	- **Subject/principal** - an active entity that can be assigned permissions e.g., user or process
		- Principal = entity that can be granted access to objects or can make statements affecting access control decisions (user account)
		- Subject = active entity within a system performing an action. e.g., process running under a principal; object in context of permissions. 
	- **Object** - resource (file, memory, device)
		- Two options for focusing control:
			1. What a subject is allowed to do.
			2. What may be done to the subject.
	- **Access operation** - RWX
		- Read - simply viewing
		- Write - includes changing, appending, deleting (integrity)
		- Execute - can run a file without knowing its contents. 
	- **Reference monitor/Security Kernel** - grants or denies access
	![](Pasted%20image%2020260225044649.png)
	Essentially the same as the Access Control Matrix model.


### Ownership
- Two access control philosophies:
	1. **Discretionary Access Control**
		- The resource owner controls who gets access, and what operations this concerns.
		- Used in most consumer systems.
		- Flexible but weaker security.
	2. **Mandatory Access Control**
		- Centralised system/network wide policy
		- Object owner's cannot override policies.
		- Used in stricter context e.g., military.
- Unix originally implements DAC, while Linux Security Modules enable MAC. 

### Unix Access Control Model
- Unix simplifies access control to three categories, with each individual object/subject receiving 9 total permission bits.
	1. **User -** file owner ('u')
	2. **Group** - assigned group ('g')
	3. **Everyone else** - ('o')
	`-rw-r----- 1 alice staff report.txt`
- Users with similar access rights can be collected into groups, and groups are given permissions to access objects on a per-object basis. 


### Linux Architecture and Security Enforcement
- Layered separation of concerns
	- Hardware, kernel space (IPC, network subsystem, security subsystem, virtual file system, process management and scheduling, system call interface), user space(user applications).

#### UID/GID
- Usernames in $*nix$ are soft aliases, the UID is what determines permissions, thus changing username would never alter privilege. 
	- User identities: UID
	- Group identities: GID
- Root has UID 0, this is hardcoded into the kernel at multiple points.
- These UIDs are stored in `/etc/passwd`
- `/etc/passwd` has/had the following structure
	- `username:password:UID:GID:Gecos:home:shell`
		- `Gecos` is a legacy optional comment field usually containing comma-separated subfields to provide descriptive information about the user. 
	- As mentioned in a prior lecture, password hashes were moved to a 'shadow file' `/etc/shadow` which is only readable by those with elevated privilege to root, to protect against offline-cracking by normal users, reducing attack surface. 

#### Root
- UID 0 is extremely powerful; it is the ultime superuser account/privilege level with unrestricted, system-wide privileges to:
	- Read/write any file
	- Kill any process
	- Change system/kernel settings `sysctl`
	- Access devices and unmount/mount file systems
- Root cannot bypass kernel-level security (SElinux, Apparmor) which can limit root regardless, cannot bypass disk encryption without the key, cannot modify or delete files marked with the immutable attribute until that attribute is removed, etc. 
	![](Pasted%20image%2020260225052557.png)
	Assignment, not comparison, gives instant privilege escalation to root. 

#### Root Management
- We protect root aggressively; if someone becomes root the entire system is compromised. 
- Write protect `/etc/passwd` and `/etc/group`
- Separate superuser duties, creating specialised system accounts e.g., `daemon`, `mysql`, follows least privilege, if web server is compromised for example, get less privileges than root.
	- Follows principle of least privilege - security concept where users, processes, and systems are granted only the minimum access rights and permissions necessary to perform their legitimate tasks.  
- Never operate as root - use temporary privilege escalation where necessary via `sudo`
- Audit `su` (switch user) and `sudo` usage to detect abuse/manipulation patterns and root actions.
### Objects
- In $*nix$, regular files, directories, devices `/dev/sda`, pipes, sockets, etc are all represented as **files** in a single hierarchal tree
- This means that access control is unified and consistent; applies uniformly across resources.
- A filename is just the label for a resource - the inode is the real object.
	- Each **inode** stores:
		- Owner(UID)
		- Group(GID)
		- Permission bits
		- Timestamps - change and b
		- File type
		- Link count
		- Pointers to data blocks
	- Security decisions are made based on UID, GID, and permission bits. 


