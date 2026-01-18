## Compiler definition
Program which translates a program written in one language into a program written in another language. 

Vast system; many internal, varying components and algorithms. Compilers which target other high-level languages rather than the instruction set of a CPU, called source-to-source/transpilers. 


### Compilers further
Microcosm of comp sci:
- **Greedy algorithms**
- **Heuristics**
- **DP**
- **Automata theory**
Deals with problems like spatial/temporal locality, memory hierarchy, pipelines scheduling.


### Properties of compilers
***The compiler must preserve the meaning of the program being compiled***
	Correctness, fundamental, must faithfully implement meaning of source code, in target lang.
***The compiler must improve the input program in some discernible way***
	Improves by making directly executable on some target machine, other compilers, improve input in different ways e.g., optimising program, targeting lang with greater ability to be executed e.g., bytecode.

### Compiler Structures
#### Single-box
![](Pasted%20image%2020260117211034.png)
- Front end
- Backend

#### Three-Phase Compiler
- Front end
- Middle-end optimiser
- Backend


### Compiler components
- **Front end**
	- Focuses, understanding, source-language program. Encode its knowledge, in some structure for later use by backend/optimiser, termed intermediate representation(may be staged), becomes definitive representation for code being translated. Can have multiple IR, but one definitive at any given stage.
- **Back end**
	- Responsible, mapping IR to instruction set + resources of target machine, backend only processes IR, assumes no syntactic/semantic errors
- **Optimiser**
	- Applies transformrations to IR to yield a semantically equivalent IR program to pass to input to the backend (unless the specific optimisation requires specific backend knowledge). Divided into passes, trade off with compilation time+compiler complexity.


