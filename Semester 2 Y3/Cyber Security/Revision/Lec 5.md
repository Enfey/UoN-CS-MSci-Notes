# Reference monitor
- Access control concept. Refers to an abstract machine that mediates access to all objects by subjects.
- Three properties:
	1. Must be **tamper proof** - can't modify 
	2. Must **always be invoked** when access is required
	3. Must be **small enough** to be **verifiable**/subject to analysis. Ensure correctness.

Functions as abstract guide of access control regarding objects; concrete details depend on security level, roles etc. 

### Security Kernel
- The security kernel is the implementation of the reference monitor
- Located in OS kernel space usually (ring 0)
- Enforces access control checks and syscall mediation. 

## Placement
- Can be placed anywhere within system - mediating access control at different levels.
- **Hardware**
	- CPU privilege levels, memory protection via MMU
	- Mediate access to physical memory, hardware devices, provide page-level memory isolation(typically in tandem with kernel)
- **Kernel**
	- Linux Security Modules
	- Windows Security Reference Monitor. 
	- Mediate access to syscalls, file access, memory, procss creation, IPC etc.
- **Hypervisor**
- **Services layer**
	- Sits above OS kernel e.g., browser sandbox, JVM runtime
	- Mediate file/network/API restrictions e.g., java code restricted from accessing local files.
- **Application layer**
	- DB access control, API authorisation checks, mediates business logic and user permissions. 
	- Very fine-grained, but often complex
- Each relies on the security kernel of the layer below e.g., DB request, then has runtime request if authed, then syscall to write, then pulls into memory, writes, pushes back. 
- It should be noted, **firewalls are not true security kernels** as they may not mediate all traffic paths, and they may not be small or verifiable. 

### Lower layers = better
- A RM or other security features at lower layer means:
	- Higher degree of security
	- Simplifies enforcement
	- Reduced performance overheads
	- Fewer layer-below attack possibilities by minimising dependencies. 
- We lose ability for fine-grained controls by restricting to blanket solutions e.g., kernels so we use layered enforcement, each layer compensating for weaknesses of others. 

### OS integrity
- OS arbitrates access requests. RW files, alloc memory, RW devices.
- OS is also a resource; accessed by user programs
- Integrity problem:
	- User-space code must be able to invoke OS functionality, but must not be able to **modify the OS** or bypass its control. 
- We must implement mechanisms to prevent untrusted code from altering the kernel security kernel. 


### Modes of Operation
- Mechanism which define which actions are permitted in each mode.
- CPU distinguishes computations done by:
	- User(untrusted)
	- OS(trusted)
- Enforced via current privilege level (CPL) bit on chip. 
- May be expressed as rings, where intermediate rings where used historically, but now just use ring 0, and ring 1 (kernel and user).
- Modes define permitted actions:
	- **Executing privileged instructions**
		- Instructions that halt CPU, change interrupt settings, configure CPU values e.g., `ptrace()`
	- **Accessing hardware directly**
		- Devices mediated via drivers + kernel subsystems
	- **Modifying page tables**
		- Critical because control of page tables means control of what memory is accessible. 

### Controlled invocation
- Allows execution of privileged instructions safely, before returning to user-space code.
- Many functions held at kernel level, buut are quite reasonably called in user space e.g., network and file I/O, memory allocation etc.
- Achieved by presenting a carefully designed interface: system calls
	1. **User executes system call**
	2. **CPU switches to kernel mode**
	3. **Kernel executes trusted code**
	4. **Returns to user mode**
	Swaps stacks. 

### Controlled Invocation: Interrupts
- Mechanism that permits controlled invocation. 
- An interrupt is a signal sent to the CPU that causes the following to occur:
	1. CPU finishes executing the current instruction. 
	2. PCB goes onto stack.
	3. Identify **interrupt source**, look up address of corresponding ISR, **provided in the interrupt descriptor table.** 
		- Descriptors hold info on crucial system objects like kernel structure locations
		- Each descriptor has descriptor privilege level, check CPL exceeds it
		- Descriptors are indexed by selectors - the interrupt number set on the CPU
	4.  Switch execution to ISR location and switch to kernel mode.
	5. Restore PCB
	6. Resume execution

### Interrupt gates
- x86 CS register contains 2 bits encoding the CPL
- Interrupt gates are descriptors inside the Interrupt Descriptor Table
	- A descriptor contains:
		- DPL
		- Gate type
		- Target handler address
	- Installed onto IDT, and used as mentioned prior to transfer execution to a more privileged level
- A **gate descriptor** is a type of descriptor used to control transfers of execution, providing controlled entry points into code at another privilege level.
	- Not all interrupts/descriptprs necessitate privilege escalation to execute the code that they point to. 

## Privilege escalation in x86 Linux
### Old Linux
- Linux, initialises interrupt descriptor table to handle syscalls at entry `0x80`; at address `0x80` it contains the descriptor for the functionality that handles syscalls.
	- CPU will verify DPL, and allows CPL = 3 to call it
	- Calls int [0x80] - the interrupt
	- CPU pushes PCB onto stack
	- CPU looks up `IDT[0x80]`
	- Verify DPL and CPL, allows CPL 3 to access IDT descriptor.
	- Jumps to handler address referenced at `IDT[0x80]`
	- Switches to kernel stack
	- Changes CPL to 0 
	- Executes `syscall`

### New linux
- No longer uses `int 0x80`, opts for `syscall` instruction on the CPU, no longer use the IDT for syscalls. 
- Instead have model specific registers, storing kernel entry point address, segment selectors for privilege switch, amongst others. 
- E.g., MSR_LSTAR contains kernel's `syscall` entry address, but basically does the same thing as before just with a syscall table. 

CALL SYSCALL WITH USERSPACE, THEN HAVE SYSCALL ENTRY ADDRESS IN MODEL SPECIFIC REGISTER, SWITCH CPL, SWITCH RIP TO MSR_LSTAR AND SAVE PCB, SWITCH STACK, LOOKUP SYSCALL HANDLER, CALL FUNCTION, RETURN TO USER MODE.

no longer use IDT here, no interrupt gates, dedicated for syscalls.



### Patching the kernel and rootkits
- If attacker gains ring 0 execution e.g., vulnerable driver they can:
	- Modify syscall handlers
- Breaks OS integrity, as kernel RM being compromised collapses all higher-layer security
- We must protect the kernel and boot and runtime.

### Secure boot
- We must ensure that the kernel is genuine and untampered with before we rely on it 
- **Secure boot** operates on the principle of chain of trust:
	- Firmware -> Bootloader -> Kernel -> Kernel modules
		Each stage verifies the cryptographic signature of the next stage prior to executing/transferring control to it. 
		Verifies drivers, kernel code, memory access code you name it
- This provides integrity at load, ensuring only trusted code is loaded at boot. 
- Doesn't automatically prove the system remains uncompromised e.g., compromised private key on side of driver author, signs malicious driver and syscall handler.

### Trusted Platform Module
- The **TRUSTED PLATFORM MODULE** is a tamper resistant chip on the CPU.
- It stores **security keys** and can perform cryptographic operations
- Provides a degree of provability/authenticity to OS + applications; records and proves what booted (used w/ secure boot, is called measured boot).
	- JUST RECORDS WHAT RAN.

#### TPM recording the boot trace
- The TPM maintains **Platform Configuration Register(s)** that record the trace of loaded components
- Each component is hashed, and PCRs maintain hash values, where the new hashes are dependent on the prior hash in the chain
	- $PCR_{new} = H(PCR_{old} ∥ H(component) )$ 
		- This is due to memory limits, and also prevents state resetting e.g., malware running then resetting back(in cases where hashing is independent)
		- Changing even 1 bit will result in different value held in PCR.
- The TPM may sign PCR values with a key $sk$ that never leaves the chip
	- Allows a verifier to check the machine booted into expected, trusted state. This is essentially compacted secure boot.


### Processes, Threads, Protection
- **Process** = program in execution with its own address space; communicates with other processes via IPC
- **Thread** = strand of execution within process that shared process address space
- Each process maintain own separate virtual address space
- OS mediates IPC
- Compromise to one process should not give access to others:
	- **Confidentiality** - prevent reading other process' memory
	- **Availability** - prevent modification of other process' memory/state
	- **Integrity** - crashes confined to one process

### Memory protection
#### Segmentation
- Split memory into logical segments to support bounds checks
- Difficult to manage at scale; hardware segementation mostly phased out.
#### Paging
- Divide memory into fixed same frames and corresponding virtual pages
- TLB cache; + MMU, invalid access triggers a page fault, faulted in from disk
- May have attack vector at paging disk
- Less good for access control, really only have permissions bits on certain pages; but things like rowhammer can sidestep entirely
- Common design: map kernel into upper portion of address space, mark as supervisor only, hardware masks and checks page permission bits. 

### Memory protection






