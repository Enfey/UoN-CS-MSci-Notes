# Exploits
 - An exploit is a technique (often code + precise input) that trigger a software or hardware bug, that is used to circumvent the operating system's security perimeter.
	 - **Bug** - defect/mistake in code
	 - **Vulnerability** - a bug with security impact(CIA)
	 - **Exploit** - a method to reliably use the vulnerability to do something security relevant e.g., crash, read memory, execute code.
	 - **Payload** - what is done after the exploit works e.g., spawn reverse shell, install malware
- Security perimeter = access control, proces isolation, privilege boundaries.

## Gaining access to user
- Social engineering, phishing. 
- If can can convince user to run, straightforward to do something malicious without any exploit, especially if grant permissions
	- Sometimes after install need to enact local privilege escalation or escape sandbox; this will take the form of an exploit. 


## Memory-Mangement bugs leading to exploits
- Memory-unsafe languages, give programmers, direct control over memory alloc + copying
- No bounds safety, leads to memory corruption bugs:
	- **Buffer overflow**
		- Writing past end of allocated array/buffer - extra bytes go into adjacent memory
	- **Stack overruns**
		- Buffer overflow in stack memory
	- **Heap overflow**
		- Buffer overflow in dynamically allocated memory, can corrupt heap objects/metadata.
- Managed memory languages reduce/eliminate classic overflow risks, enforce bounds checks and/or safe ownership, can still cause vulnerabilities, just different class of bugs. 


### Buffer Overflows
- A buffer is a contiguous block of memory e.g., `char[]`
- When a program writes more data than buffer can hold, get **buffer overflow**
- Memory next to buffer, often contains other important data; e.g., on stack, may be dynamically allocated, may be in .data, could be on stack if non-static.
- Overwriting the data next to it can change program behaviour, leak secrets, and potentially lead to full compromise.

### Memory layout
- Memory stored in virtual address space from 0x0 to 0xFFFFFFFF for 32 bit
- Typically depends on object file format and loader determining the concrete sections a program will have available; segments separated to manage permissions separately to avoid WX page allocation. 
	![](Pasted%20image%2020260226043755.png)
	The generic model.
RO page mapping, ASLR, NX Stack etc. 


### The stack
- Holds info about function calls growing downward from the top of the virtual address space.
- Stores:
	- Return address, saved registers, local variables, saved frame pointer(as the current one is overwritten), params.  
- Accesses local storage from via frame pointer. 
	1. Function call pushes new frame onto stack, all required info, set up local variables, params etc.
	2. Function epilogue/postcall, the frame pointer is restored, and a return will pop the return address of the stack, and it will be jumped to.
	![](Pasted%20image%2020260226044413.png)
- Now, if a bug lets you overwrite `ret` for a given stack frame that will eventually be returned to, you can redirect execution to somewhere within the process when that function returns. 


### Stack smashing
- Technique, exploit buffer overflow vulnerability to write past end of stack-allocated buffer, and corrupt the stack frame. 
- The basic form of stack smashing involves malicious software writing new opcodes into the buffer, then attempting to execute that written memory
	- The overflow would overwrite the return address so control flow jumps to the malicious instructions. 
	- Due to NX stack, not usually employed.
- The alternative form of stack smashing involves filling buffer with contents and then providing an alternative return address, so when the function executes `ret` the CPU jumps to an address chosen by the attacker - does not require the stack itself to be executable.
	- just use e.g., strcpy, performs no bounds checking, write long string into memory in hopes of overwriting prior stack frame return address to chosen address.

### Stack hardening - Stack Canaries
- These are values placed on the stack to detect overwrites
- Canaries modify the function prologue/epilogue to check a value before returing
- The stack frame has a *CANARY* value $CY$ placed in front of the return address. 
	![](Pasted%20image%2020260226051451.png)
- On function entry, compiler stores a pseudorandom canary value in stack frame
- Before returning, determine whether canary has been changed. 
- If the canary has changed, the program aborts, preventing corrupted `ret` from being used. 
#### Limitations
- If attacker can leak memory, they can just place the canary value in the buffer. 
- Some overflows, target data that doesn't cross the canary e.g., data attacks. If can modify local variable and guess logic at that part of the program, then can potentially craft another exploit. 
- Attackers can sometimes bruteforce CY but it is rare(its newly generated, what a shit idea)

### Stack Hardening - NX Stack
- Modern OS mark the stack pages as NX
- The same may also be the case for the heap.
- Means injected bytes via a buffer overflow cannot be executed as code.
- If stack pages are NX; putting shellcode on the stack and attempting to execute it e.g., overwrite return addr too will not work as the CPU will fault; is NX.
- On Linux, specify NX stack via PT_GNU_STACK segment in an ELF object which can be loaded.

## Ret2libc + ROP
### Ret2libc
- Instead of injecting new code onto the stack, attacker points overwritten return address to function mapped in memory, often `libc`, almost always mapped RO. 
- We already have powerful code in process address space; control IP via `ret` overwrite, circumventing NX stack, we can jump to that code.
- Most famously, jump to `execve()` but `libc` has routines for pretty much anything. 
	- If orchestrated carefully, can achieve arbitrary code execution. 
	- If can control the arguments, that is. 
		- After control transferred to start of function via `ret` you can make stack contain controlled bytes that look like legit arguments. 
		- Though this depends on calling conventions, x86-64 attackers need to load values into argument registers before transferring control, which is more complicated as this requires execution.

### ROP
- Return-oriented programming = code-reuse exploit technique that enables attackers chosen computation in face of stack/heap marked NX pages
- If the attacker can corrupt control flow, most commonly by overwriting saved return address, they can redirect execution into existing executable code mapped in memory. 
- A **gadget** is a short instruction sequence present in memory that ends in control-transfer, classically `ret`
- Compilers generate many gadgets naturally. 
- The attackers goal is to smash the stack via an overflow, overwriting a large stretch of the stack to contain a sequence of gadget addresses such that repeated `ret` instructions walk through that sequence. 
	- We do not return to callers next intended instruction.
	- We overwrite the saved return address in current function's frame so it jumps to the gadget during epilogue. 
	- The first ret jumps to the first gadget, and that gadget's ret returns to the next address (top of the stack after the current frame is evicted, and so on and so forth).
- We can combine ROP with ret2libc; we use gadget chaining to set up register contents(where params are expected in particular registers)
	- Generate string, then overwrite return address with execve() and call with that param, can be carefully constructed.
- Cannot be performed if can't write far enough to control return address or if stack canaries exist.

#### x86 instructions and registers
- **EBP** - each call sets new one, fixed in frame, used to access locals
- **ESP** - holds address pointing to top of stack
- **EIP** - $PC$
- **ret** - reads 4 byte value at top of stack, pops off into $EIP$, adds 4 to $ESP$ popping that value off stack. 
- So in ROP, chain gadgets [2, 3...n] of the next stack frame(s) up to $n$ and then return address of the immediate frame is set to the address of gadget 1.
	- One would also load parameters for any `ret2libc` exploit, and the `ret2libc` address after `n`.

### General hardening
- **ASLR**
	- Randomise base address of loaded object, and perhaps, its constituent segments.
		- PIE binaries get completely random base, non PIE linked with fixed base address in mind, so only libs/stack/heap move
	- User stack placed near top of VA space, with randomised offset
		- Many kernels also randomise stack initialisation details
	- Starting point of heap may be randomised
	- In doing this, control-flow hijacks like those described above are made more difficult; they rely on known code addresses
	- Hardens; must leak a pointer to recover the randomised base and then compute gadget/function addresses accordingly.
- **ASCII armouring/null bytes**
	- ASLR+alignment+changing addresses may force as useful function or gadget to live at an address containing 0x00, null byte
	- Using function like `strcpy()` the ROP chain would be terminated early, preventing the exploit, sees NUL-terminator
		- But can just use alternative gadgets/functions e.g., exec has a whole family. 
- **Restricting access to obvious syscalls**
	- Even if attacker gains control of instruction flow, an reduce impact by limiting what the process is allowed to do. 


## Concurrency bugs and exploits
- In multitasking environment, potential for multiple threads of execution to have been scheduled in manner, that breaks programmer assumption.
- With concurrency, the classic pattern is TOCTTOU
	- E.g., between opening file and writing it to a buffer, attacker symlinks the file to elsewhere. 
- Common exploit outcomes from TOCTTOU bugs include privilege escalation, data leakage/corruption, and policy bypass; e.g., race condition to change privilege of user in OS or in some system, attacker links their name, and their privilege is changed during the time of use. 
- The general rule for preventing this class of attacks:
	- Atomic operations
	- Do not validate and then user later without an additional check or lowering privilege. 


