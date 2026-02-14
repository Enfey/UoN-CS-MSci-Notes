# Reference Monitor
> An access control concept that refers to an abstract machine that mediates all access to objects by subjects

Three essential properties:
- Must be **tamper proof/resistant**. 
- Must **always** be invoked when access is required.
- Must be small enough to be **verifiable**/subject to analysis to ensure **correctness.**

Functions as generic/abstract guide of access control regarding objects in terms of subjects (e.g., users, processes), with concrete details depending on placement and role. 

All accesses must pass through it concerning the guarded objects, must be unmodifiable by untrusted code, and must be simple enough to formally reason about.

## Security Kernel
- The security kernel is the *implementation* of the reference monitor
- Typically located in OS kernel space (ring 0)
- Enforces:
	- Access control checks.
	- System call mediation.
## Placement
- Can be placed anywhere within a system
	- Hardware
		- CPU privilege levels and memory protection via MMU
		- Mediate access to physical memory, hardware devices, can provide page-level memory isolation(typically in tandem with kernel)
	- Kernel
		- Windows Security Reference Monitor (SRM), Linux Security Modules (LSM). Most common placement.
		- Mediate access to syscalls, file access, memory, process creation, tracing processes, IPC, networking operations.
		- Facilitated in part by prior layer.
	- Hypervisor
	- Services layer
		- Sits above OS kernel e.g., browser sandbox, .NET sandbox, JVM security manager, container runtimes
		- Mediate file/network/runtime API restrictions within the environment. 
		- E.g., java code may be restricted from accessing local files. 
	- Application layer
		- DB access control, API authorisation checks. 
		- Mediates business logic and user permissions e.g., user $x$ can ready record $y$ (not in the OS context). 
		- Very fine-grained, but often complex, and susceptible to logic bugs. 
- It should be noted, firewalls are not true security kernels as they may not mediate all traffic paths, and they may not be small or verifiable. 

### Why lower layers are better
- An RM or other security features at a lower layer usually means:
	- Can assure a higher degree of security
	- Simplifies enforcement
	- Reduced performance overheads
	- Fewer layer-below attack possibilities by minimising dependencies.
- We lose contextual awareness and potentially the ability for fine-grained control by just restricting to blanket solutions e.g., kernels, as these are widespread
- Thus, the solution is to use layered enforcement, with each layer compensating for the weaknesses of others. 
	$Hardware → Kernel → Application$



## OS integrity
- The OS arbitrates access requests e.g., who can read/write files, allocate memory, access devices
- But the OS is also a resource that must be accessed by user programs
- We arrive at an integrity problem:
	- User-space code must be able to invoke OS functionality, but must not be able to modify the OS or bypass its control.
- If we do not provide mechanisms to prevent this, then untrusted code could alter the kernel level enforcement mechanism itself. 
### Modes of Operation
- Modes of operation are the mechanism which define which actions are permitted in which mode
- The CPU distinguishes computations done on behalf of:
	- The OS (trusted)
	- The user(untrusted)
- This is enforced by a privilege mode bit/current privilege level (CPL) on the chip.
- This may be expressed as rings, where intermediate rings where used historically, but now just use ring 0 and ring 1 (kernel and user, respectively).
- Modes define which actions are permitted, especially:
	- Executing privileged instructions
		- E.g., instructions that halt the CPU, change interrupt settings, configure process/CPU values e.g., `ptrace()`
	- Accessing hardware devices directly
		- Devices mediated via drivers and kernel subsystems.
	- Modifying page tables/memory mapping
		- Critical because control of page tables means control of what memory is accessible. 

### Controlled Invocation
- Allows the execution of privileged instructions safely, before returning to user-space code.
	- Many functions held at kernel level, but are quite reasonably called in user-space
	- Network and file I/O, memory allocation(though often not the case for POSIX-compliant systems) etc. 
- Achieved by presenting a carefully designed interface: system calls
	1. User executes system call
	2. CPU switches to kernel mode
	3. Kernel executes trusted code
	4. Returns to user mode.
- Prevents arbitrary jumps from user to kernel mode, restricts entry points, swaps stacks (kernel stack).

### Controlled Invocation: Interrupts
- Mechanism that permits controlled invocation. 
	- In many ways, equivalent to software exceptions.
- They are the mechanism for controlled transfers of control; system calls historically used software interrupts to request kernel services. 
- Given an interrupt which signals the CPU:
	1. CPU finishes executing current instruction
	2. Put PCB on stack (or just specific elements)
	3. Identify interrupt source and look up the address of the corresponding ISR, which is provided in the **interrupt descriptor table**
		**Descriptors** hold information on crucial system objects like kernel structure locations.
		Each descriptor has a descriptor privilege level, CPU checks that $CPL \le DPL$ to protect kernel.
		Descriptors are indexed by selectors (interrupt number set on CPU)
	4. Switch execution to ISR location and switch to kernel mode
	5. Restore state
	6. Resume execution.
	![](Pasted%20image%2020260213005258.png)

#### Interrupt-Gates
- x86's CS register contains 2 bits encoding the **CPL**
- Interrupt gates are descriptors in the **Interrupt Descriptor Table**
	- A descriptor contains:
		- Target handler address
		- DPL
		- Gate type
		- And some more
	- Installed into the IDT, and are used to securely transfer execution to a more privileged level. 
- What we mean is that a **gate descriptor** is a type of descriptor used to control transfers of execution, providing controlled entry points into code at another privilege level. 
	- Not all interrupts necessitate privilege escalation.

### Privilege escalation in x86 Linux
#### Old Linux
- Linux initialises its interrupt descriptor to handle syscalls at entry `0x80`.
- When `int $0x80` after compilation/linking executes:
	- CPU verifies DPL, allows CPL = 3 to call it
	- CPU:
		- Pushes user state onto stack
		- Switches to kernel stack
		- Changes CPL to 0
	- Jumps to handler address referenced at `IDT[0x80]`
	- Executes `syscall`
	- Returns with `iret`
![](Pasted%20image%2020260213010959.png)
![](Pasted%20image%2020260213011010.png)


#### New Linux
- No longer uses `int 0x80` by default, instead opting for a `syscall` instruction on the CPU, and no longer use the IDT. 
- Instead opt for **model-specific registers**
	- Modern CPUs have these, store kernel enty point address, segment selectors for privilege switch, amongst otherts
	- Only writable in ring 0
- When compiled, typically compiles such that the number tied to the user-space `syscall` wrapper is moved into register `RAX`
- Then execute the `syscall` instruction:
	- Which switches the CPL
	- Switches $RIP$ to $MSR\_LSTAR$(model-specific register containing the kernel's `syscall` entry address).
	- The entry point for $LSTAR$ performs:
		- Saves the user return state e.g., old $RIP$
		- Switch to kernel stack
		- Validate syscall number
		- Lookup handler in syscall table
		- Call appropriate function
	- Returns to user mode as standard by restoring $RIP$ and returning CPL etc.


Differences:
- Old uses IDT, not used here
- Old uses interrupt gates, not used here
- Hardware stack switch vs entry stub handling switch
- Dedicated for syscall's.

## Patching the kernel and rootkits
- If the attacker gains ring 0 execution e.g., via vulnerable driver, they can
	- Modify syscall handlers
	- Install pre-hooks etc
- Breaks OS integrity, as the kernel reference monitor being compromised collapses all higher-layer security.
- We must protect the kernel at boot and runtime.
### Secure Boot
- We must ensure the reference monitor/kernel is genuine before we rely on it - thus we utilise secure boot. 
- Secure boot operates on the principle of **chain of trust**:
	- $Firmware → Bootloader → Kernel → Kernel \ modules$ 
	- Each stage verifies the **cryptographic signature of the next stage** prior to executing/transferring control to it.
- Provides integrity at load, ensuring only trusted code is loaded at boot. This doesn't automatically prove the system remains uncompromised, but it does prevent complete subversion of the enforcement logic.

### Trusted Platform Module (TPM)
- Trusted Platform Module is a tamper-resistance chip (on the CPU) that stores **security keys**, and is capable of performing some cryptographic operations
- Provides a degree of provability/authenticity to OS and applications, as it records and proves what booted in collaboration with secure boot. 

#### TPM recording the boot trace
- The TPM maintains Platform Configuration Registers (PCRs) that record the trace of loaded components. 
- Each component is hashed, and PCRs are extended with the prior hash value
	- $PCR_i = H(PCR_{i-1} ∥ H(component))$ 
	- Thus a modification to any modules changes all subsequent PCRs, can verify exactly where compromise occurred in case of threat/attack remediation. 
- The TPM can sign $PCR$ values with a key $sk$ that never leaves the chip
	- The signed quote permits a verifier to check whether the machine booted into an expected, trusted state.
	- Or verify otherwise :0

### Processes, Threads, and Protection
- **Process** = program in execution with its own address space, communicates with other processes (IPC) via the OS
- **Thread** = strand of execution within a process that shares the process address space
- Each process maintains its own separate virtual address space
- OS mediates IPC
- Compromise of one process should therefore not automatically give access to others:
	- **Confidentiality:** prevent reading other process' memory
	- **Integrity**: prevent modifying other processes' memory/state
	- **Availability/fault containment:** crashes confined to one process
- Threads are more difficult; thread-level isolation is weak by design.


### Memory protection
#### Segmentation
- Split memory into logical segments to support bounds checks
- Complex to manage at scale
#### Paging
- Divide memory into fixed size pages with frame mapping, TLB cache, invalid access triggers a fault.
- May have attack vector at paging disk.
- Less good for access control; really only have permission bits.
	- Common OS design; map kernel into upper portion of virtual address space, but mark kernel pages as supervisor only, such that user mode cannot access it, hardware masks and checks page permission bits.

### Meltdown 
- Exploit that reads privileged memory use a side-channel attack despite supervisor protections
- User-mode access to supervisor pages should fault, therefore user code cannot learn memory contents
- However, CPUs speculatively execute instructions and roll back state if failed (for meltdown, concerned with illegal access), but the cache is not rolled back
- So speculative execution transiently loads a kernel byte.
	- If we can measure access time precisely, we can infer whether data is cached
		1. Flush the cache
		2. CPU speculatively executes the illegal instruction, loading a specific probe page into the cache
		3. CPU realises illegal access(MELTDOWN) and rolls back, but the page remains in the cache
		4. Read all pages pointed to, and time them, if extremely fast, then know address in cache (will be grabbed from there).
- Can then choose target address via secret val (data), flush the cache, run the exploit to load specific address into cache, realise illegal access and roll back, and can then mask out single supervisor bit to access as if it were user memory.
![](Pasted%20image%2020260213024421.png)
![](Pasted%20image%2020260213024425.png)

### Spectre
- Similar to metltdown, speculative execution utilised to skirt around application bounds checks.
- Speculatively:
	1. CPU enters branch
	2. Reads OOB memory
	3. Uses secret value to index probe table
	4. Caches specific page
	5. Later detects **misprediction**
	6. Rolls back
- Mask out single bit, accessing user memory at that location.

![](Pasted%20image%2020260213024430.png)

