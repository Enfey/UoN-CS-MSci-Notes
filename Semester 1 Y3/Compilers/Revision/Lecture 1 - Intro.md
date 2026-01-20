## Compiler definition
<mark style="background: #FFF3A3A6;">Program which translates a program written in one language into a program written in another language. 
</mark>
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
	- Focuses, u<mark style="background: #FFF3A3A6;">nderstanding, source-language program</mark>. <mark style="background: #FFF3A3A6;">Encode </mark>its knowledge, in <mark style="background: #FFF3A3A6;">some structure</mark> for later use by backend/optimiser, termed intermediate representation(may be staged), becomes definitive representation for code being translated. <mark style="background: #FFF3A3A6;">Can have multiple IR</mark>, but <mark style="background: #FFF3A3A6;">one definitive</mark> at any given stage.
- **Back end**
	- Responsible, <mark style="background: #FFF3A3A6;">mapping IR</mark> to <mark style="background: #FFF3A3A6;">instruction set + resources of target machine</mark>, backend only processes IR, <mark style="background: #FFF3A3A6;">assumes no syntactic/semantic errors</mark>
- **Optimiser**
	- Applies<mark style="background: #FFF3A3A6;"> transformrations</mark> to <mark style="background: #FFF3A3A6;">IR</mark> to <mark style="background: #FFF3A3A6;">yield </mark>a <mark style="background: #FFF3A3A6;">semantically </mark><mark style="background: #FFF3A3A6;">equivalent</mark> <mark style="background: #FFF3A3A6;">IR</mark> program to pass to input to the backend (unless the specific optimisation requires specific backend knowledge). Divided into passes, <mark style="background: #FFF3A3A6;">trade off with compilation time</mark>+<mark style="background: #FFF3A3A6;">compiler complexity.</mark>


