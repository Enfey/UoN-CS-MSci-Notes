## Role of the OS
- An OS combines:
	- **Resource/device management** - CPU, mem, stroage, drivers
	- **Identification + authentication**
	- **Access control/authorisation** - permit access to objs according to principal access rights
	- **Auditing** -logging actions
	- **User account management**
	- **Configuration**
- The OS security components, act, as security kernel.


## Authentication + Authorisation
- Assuming authenticated, system must enforce authorisation. 
- The core components:
	- **Subject/Principal**
		- Active entity that can be assigned permissions.
		- **Principal** = entity granted access to objs, or can make statements affecting access control decisions
		- **Subject** = active entity within system, performing an action e.g., process running under a principal
	- **Object**
		- Resource
		- Two options:
			1. What a subject is allowed to do with object
			2. What may be done to the subject by the object
	- **Access operation**
		- RWX
			- X = execute without knowing contents
	- **Reference monitor/security kernel**
		![](Pasted%20image%2020260225044649.png)


### Ownership
- Two access control philosophies:
	1. **Discretionary Access Control**
		- Resource owner controls who gets access, and what operations principals can perform
		- Used in consumer systems; flexible, but weaker sec.
	2. **Mandatory Access Control**
		- Centralised system/policy
		- Object's owner cannot override policies
		- Used in stricter contexts
- Unix originally implements DAC, LSM enable MAC. 


### Unix Access Control
- Unix simplifies to three categories, with each individual object/subject receiving 9 total permission bits. 
	1. **User** - file owner ('U') 
	2. **Group** - assigned group ('g')
	3. **Everyone else** - ('o')
	`-rw-r----- 1 alice staff report.txt`
- Users w/ similar access rights collected into groups.
- Per-object basis, similar to DACLs, specify RWX for owner, group, and everyone else. 

### Linux Architecture and Security Enforcement
- Layered separation of concerns:
	- Hardware, kernel space (IPC, networking, security subsystem, file system syscall interface), user space (user applications).

### UID/GID
- Usernames are aliases in $*nix$.
- The UID is what determines permissions:
	- UID - user identities; GID - group identities
- Root has UID 0, hardcoded into kernel.
- UIDs stored in `/etc/passwd`
- `/etc/passwd` has/had the following structure:
	- `username:password:UID:GID:Gecos:home:shell`
	- Password hashes were moved to a shadow file `/etc/shadow` which is only readable by those with elevated privilege to root to protect against offline-cracking attacks. 

### Root
- UID 0 is extremely powerful; unrestricted, system-wide privilege to:
	- Read/write any file
	- Kill any process
	- Change system/kernel settings via `sysctl`
	- Access devices
- Cannot bypass kernel security which can limit root regardless, can't bypass disk encryption, can't modify or delete files marked with the immutable attribute until that attribute is removed, etc. 


### Root management
- We protect root aggressively; if someone becomes root, the entire system is compromised. 
- Write protect `/etc/passwd` and `/etc/group`
- Separate superuser duties via sep accounts e.g., `daemon`, `mysql`, following principle of least privilege e.g., if web server compromised, don't instantly get root. 
	- Grant minimum access rights + perms to perform legitimate tasks
- Never operate as root, use temporary privilege escalation via `sudo`
- Audit `su` (switch user) and `sudo` usage to detect abuse.


### Objects
- In unix, regular files, dirs, devices, pipes etc, all represented as files
- Means access control = consistent and unified.
- Filename is just a label for resource, the `inode` is the real object.
- INODE:
	- Stores:
		- **Owner(SID)**
		- **Group(GID)**
		- **Permission bits**
		- **File type**
		- **Link count**
		- **Pointers to data blocks**
	- Does not store filename; file names are stored in directories which map names to inode numbers.
	- Security decisions are made based on SID, GID, and permission bits. 

### Permissions
- Every file has **9 permission bits**.
	- User, Group, Others
	- RWX
- Held in inode metadata
- **Octal** - base 8, representation.
	- **Bit 3**
		- read(0x4)
	- **Bit 2**
		- write(0x2)
	- **Bit 1**
		- execute(0x1)
- Permissions changed via `chmod` passing three octal values and the path to file to be altered.

#### Example
- 754 for owner - can list directory contents, execute it, modify the files as they wish. for group, can read and execute, for everyone else, can only read. 

#### Directory permissions
- Different semantics to regular files
- For directories:
	- $r$ = list files *within* the directory
	- $w$ = create/delete files in the directory
	- $x$ = **traverse** directory e.g., $cd$ 
- Can have execute with read, just means can access known files, but can't list contents


### SUID
- Set user ID
- Special permission bit set on an executable
- When this bit is set, the program runs with the **effective UID of the file owner** rather than the user who started it. 
- Only the file owner or root can set the SUID bit
- Privilege escalation = concern where the file owner is root and SUID is set; can run the program as root. 
- We need SUID to allow non-privileged access to privileged actions
	- But this is dangerous - if the program contains bugs and attacker can spawn a shell, the shell will spawn at root
	- Must only enable SUID for root/admins on programs that do are known not have bugs. 

### Linux security modules
- Unix used DAC - per-file set of perms decided by file owner, validated by kernel by checking inode SID, GID, and permission bits upon access requests. 
	`User->System call->Kernel->Check inode permissions->Allow/deny`
- Linux security modules introduced **security hooks** inside the kernel which permit extra security checks on top of standard Unix DAC (centralised).
	- Hard to enforced DAC at scale, but is highlighy configurable.
	- There is no second opinion as to whether someone should be able to perform a particular action. If running as root for example, no questions asked. 
- LSM:
	- User level process makes syscall
	- Kernel looks up inode
	- Error check
	- DAC check(RWX)
	- LSM hooks in - determined according to security module, customisable
	- Access granted/denied
- The flow depends on the security module used.
- There are many different types of access control hooks for different types of objects.