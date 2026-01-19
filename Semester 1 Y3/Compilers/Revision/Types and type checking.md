## Type
> An abstract category that specifies a set of properties held in common by all its members.

Can be defined by membership e.g., defining range of values
Can be defined by rules e.g., defining structure with specific fields.

In compilers types, let us know how different values behave, and directly describes their memory footprint. Most portions of backend of compiler, need to know type of exvery expression to perform their job adequately.


## Type System
> Comprises the set of available types, the rules governing their use and interaction. Well-designed type systems increase expressiveness, and enable more efficient code emission by eliminating runtime checks. 

- Which values be.long to what types
- What types can be inferred/must be declared explicityly
- In type theory we say $e : τ$, the typing judgement, to posit that $e$ has the type $τ$. 
- Yields extra layer of semantics over grammar e.g.,
	- "abc" – 3 is a syntactically correct (binary expression applied to two literal values) but meaningless in most languages
	- var1 + var2 is syntactically correct (binary expression applied to two named variables) but its meaning depends on the type of those variables
### Type system components
#### Base types and compound types
- **Base types** are 'core types' in the language
	- **Int, float, char**
		- Can be **directly represented in memory**
		- Also called primitives
		- All other types built out of base types
- **Compound types** are built out of other types
	- Arrays, structures, enums, unions
	- Most useful languages require them.

##### Compound types: Arrays
- Contiguous sequence of elements
- Values have same type
- Have a defined size, which may or may not be part of its type
	- In C, int[x]; and int y[10]; have the same type, in fortran, they do not. 
##### Compound types: Enums
- Enumerated types are those with a predefined set of values
- Some language treat enums as the name of constants
	- E.g., C/C++, enums are aliases for integer values
	- So they are interchangeable with the base type
- Some language treat each defined enum as a distinct type
	- Each java enum is a separate class, not equivalent to any other class. 
##### Compound types: Structures
- A structure is a type made up from a collection of other types
- Allow access to each internal value, but can be manipulated as a whole 
- If a struct $S$ has fields of types $a,b,$ and $c$, we say that $S$ is the type $a \times b \times c$ 
- In type theory, this is a **product type**.
- ![](Pasted%20image%2020260119165851.png)
##### Compound types: Unions
- A union is a type that indicates a value will have one of several predefined types called its **variants.**
- Permits multiple interpretations. 
- Some unions are **tagged**; they track which variant a union currently holds.
- Some unions are **untagged**; they do not track which variant a union currently holds. 
- ![](Pasted%20image%2020260119170034.png)
- Called a sum type in type theory
- ![](Pasted%20image%2020260119170817.png)
- + if tagged, U if untagged, for a union S

##### Compound types: Functions
- Some languages allow functions to be used as values
- These languages require functions to have their own types in the type system


#### Type equivalence
- Provides the unambiguous rule necessary to answer whether two different type declarations are equivalent or compatible. 
- May be defined by a formal judgement of the form $\Gamma \vdash A = B$ in non-trivial cases. 
- Equivalence of base types obvious
- Equivalece of constructed types has to be defined carefully:
	1. **Name equivalence**
		Type are equivalent only if they share the same name
		![](Pasted%20image%2020260119171501.png)
	2. **Structural equivalence**
		Types are equivalent if and only if they have the same internal structure. Requires that types consist of same set of fields, in same order, with all those fields having equivalent types.
		![](Pasted%20image%2020260119171510.png)
#### Type Inference
- Formal logical rules that tell us how to derive the type of an expression
- Every expression in program must have an assigned type, may be assigned at either runtime or compile time. 
- We set up **inference rules** to define how to work out the type of an expression given the types of its constituents. 
- A compiler uses these inference rules to calculate the type of every expression in a program. 
- ![](Pasted%20image%2020260119171643.png)
- ![](Pasted%20image%2020260119171659.png)
- Generally refers to scope, calculate types of e_1n and e_2, then look for inference rule that matches to determine type of expression.
- If operator does multiple things/accepts multiple types, must have differing inference rules, see above slide. 
#### Type promotion
- Implicit conversion of types into another compatible type
- When dealing with mixed type expressions that would otherwise fail, the compiler may be at liberty to insert conversion code, often converting one or both operands to a more general type
- Numeric promotion, extend width of one or more operands
- Many ways to achieve, e.g., producing promotion rules in the type system for different types, which we can refer to as **subtyping.**