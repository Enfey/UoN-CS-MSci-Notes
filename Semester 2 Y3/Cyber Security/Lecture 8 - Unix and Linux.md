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
		- **Owner(UID)**
		- **Group(GID)**
		- **Permission bits**
		- **Timestamps**
		- **File type**
		- **Link count**
		- **Pointers to data blocks**
			![](Pasted%20image%2020260226014612.png)
	- Notably, it does not store the filename, filenames are stored in directories, which map nambes to inode numbers. 
	- Security decisions are made based on **UID**, **GID**, and **permission bits**. 

#### Permissions
- Every file has **9 permission bits**
	- **User - RWX**
	- **Group RWX**
	- **Others - RWX**
- Held in inode metadata
- Octal representation - base 8:
	- **Bit 3 - read (0x4)**
	- **Bit 2 - write (0x2)**
	- **Bit 1 - execute (0x1)**
- Permissions are changed using `chmod`, passing three octal values, and the path to the file to be altered.
##### Example
- Given `-rwxr-xr-x 1 sbzmpp staff 8844 Feb 28 18:20 index.html`, permissions = $755$
- Given a directory with octal permissions $754$ the owner can list the directory contents, create/delete files in the directory, and can traverse it, but group referred to by $GID$ can only list the contents and traverse it, but everyone else can only list the contents. 
#### Directory permissions
- Directory permissions have slightly different semantics to regular files
- For directories:
	- $r$ = list files within the directory (`ls`)
	- $w$ = create/delete files in directory
	- $x$ = traverse directory (`cd`, access files/directories inside).
- Can have execute without read - just means can access known files but cannot list directory contents.
- You need write+execute to delete files inside a directory, but do not need execute for creation?


### SUID
- SUID = Set User Id
- It is a special permission bit set on an executable file.
- When this bit is set, the program runs with the **effective UID of the file owner** rather than the user who started it. 
- Normally an executable looks like this:
	`-rwxr-xr-x 1 root root file`
	When SUID is set, the owner's execute bit becomes:
	`-rwsr-xr-x 1 root root file`
	SUID is set by altering the permission bits either with `chmod s+u` or `chmod 4***` where the asterisks are the other permission bits and the 4 represents that SUID is enabled. 
- Only the file owner, or root can set the SUID bit. 
- Privilege escalation is only a concern where the file owner is root and the SUID is set. 
- We need SUID to allow non-privileged access to privileged actions e.g., changing passwors
	- But this is dangerous, if the program contains bugs and the attacker can spawn a shell, the shell is at root and will have total system control.
	- Common attack vector, must only enable this on programs which do not have bugs. 

### Linux Security Modules
- Unix traditionally used Discretionary Access Control - which is a per-file set of permissions decided by the file owner, validated by the kernel by checking the *inode*.
- The typical access flow at a high level:
	- `User->System call->Kernel->Check inode permissions->Allow/deny`
- Linux security modules introduced security hooks inside the kernel which permit extra security checks after standard Unix DAC akin to Mandatory Access Control(MAC).
	- The issue with pure DAC is that while highly configurable, it becomes difficult to enforce at scale.
	- There is no second opinion as to whether someone should be able to perform something; for example if running as root, DAC almost always allows everything without further checks.
- The new flow with LSM:
	1. User level process makes system call to access file in some capacity
	2. Kernel looks up inode
	3. Error checks
	4. DAC check (traditional RWX)
	5. LSM hook -> determines via permitted according to security module
	6. Access granted or denied. 
- Note that the flow depends on which security module is active(there may be limited stacking), and depends on the specific syscall and specific operation. 

#### Example Hook
![](Pasted%20image%2020260226021643.png)


#### SELinux
> Kernel-level Mandatory Access Control system implemented using the Linux Security Module framework.

- Attaches a **security context label** to everything considered a file under $*nix$ 
	- `user:role:context type:level`
		- Type is what TE compares
		- Role is for RBAC, controls which types a user can enter.
		- User is defined custom by SELinux, and these are mapped to linux users - many map to unconfined_u.
			![](Pasted%20image%2020260226031159.png)
- These labels are stored in extended attributes $xattrs$ for files.
- It converts labels into security identifiers used inside the kernel for rapid decisions rather than comparing strings.
- When a process attempts an operation and the syscall hooks into SELinux, it receives:
	- Source SID
	- Target SID
	- Object class(file, dir, socket, etc).
	- Requested permission
- SELinux checks policy rules and allows or denies respectively. 
	- The core policy model for SELinux is usually comprised of type enforcement - a rule system that says a process of type $X$ may perform operation $Y$ on object of type $Z$.
		`allow httpd_t httpd_sys_content_t:file read;`
		Source type = `httpd_t`
		Target type = `httpd_sys_content_t`
		Object class = `file`
		Permission = `read`
	- When a process accesses a 'file', the process security context label and the target security context label has the type field extracted, and that is checked against the policy model, which essentially functions as an access control matrix. 
	- Defaults to no if there is no policy.



#### App Armor
- Installed as standard on openSUSE and Ubuntu 
- Much easier to set up than SELinux 
- As with SELinux, is usually used to enforce policies on services and daemons




