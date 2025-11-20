# Type
> A type is an abstract category that specifies a set of properties held in common by all its members.

A type can be specified either by membership e.g., defining a range of values, or by rules e.g., defining a structure with specific fields. 

In compilers, a type lets us know how different values behave and directly describes their memory footprint. Portions of a compiler backend need to know the type of every expression to perform their role adequately. 

# Type system
>Comprises the set of available types and the rules governing their use and interaction. A well-designed type system increases language expressiveness and enables more efficient code emission by eliminating runtime checks.

The fundamental relationship between a program term and its classification is via the **typing judgement** $e : τ$. This is a two-place infix relation defined by **Type Rules** which formally assert the validity of judgements based on premises. 

The judgment is necessarily extended to include a **Context**, resulting in the structure:$$Γ⊢e:τ$$
The context in our case is the static typing environment, typically an ordered list of distinct variables and their corresponding types, such that the variables are known during the typing judgement. In practice, context is realised as the variables that are currently in scope together with the types the type checker knows for them.

The validity of a judgement is established through **type rules**, which fit within the framework of formal proof systems. 

Each rule is presented with zero or more premises above a horizontal line separating premises above, and a single conclusion below. 
![[Pasted image 20251104003529.png]]
A **derivation** is a tree structure composed of judgements, where the root (conclusion) is derived from its leaves (premises) by correctly applying the rules of the formal type system. A judgement is **valid** if it can be obtained as the root of such a derivation i.e., a program fragment is deemed well-typed if and only if a complete formal proof(the derivation tree) can be constructed. 

For example, proving that $3+4 \leq 5$ has type bool involves constructing a derivation tree.

$$\frac{\frac{3:N \ \ \ 4:N} {3+4:N}  PLUS \ 5 : N}{3+4 \leq 5: bool}Leq$$

### Metatheory: Structural Properties and Consistency
Formal language definitions must establish properties about the internal consistency of typing judgments and their environments. These are often called structural properties. 

#### Weakening
> If a term $e$ typechecks in a context ($\Gamma, \Gamma' \vdash e : \tau$) it still type checks in a larger context where an unused variable has been added. This is proved by structural induction on the derivation. 
#### Exchange/Contraction
> The order of variable declarations in the context can be reordered without affecting type correctness.

## Type System Components

### Base types
> **Base types** are the fundamental data kinds supported directly by the language (and often by the underlying processor), such as integers, floating-point numbers, characters, and booleans. 

 They can be represented directly in memory. All other types are constructed out of base types.
### Compound types/Constructed Types
Compound types are types that aggregate individual types/combine types according to defined rules. 

#### Arrays
> An aggregate object that groups multiple elements of the same type, providing each element an implicit name that is computed. The array's size/dimensions is sometimes included in its type to check for incompatible operations. 



#### Enumerated Types
> A programmer defined-type consisting of a small, specific set of named constant values. The elements are inherently ordered(but in practice this is largely irrelevant). 

Some languages treat enums as the names of constants e.g., C/C++ enums are just aliases for integer values, and are thus interchangeable with the base type.

Other languages treat each defined enum as a distinct type. 
#### Structures
> An aggregate of multiple objects of arbitrary types, identified by explicit names. In type theory, this is formally defined as a **Product Type**.

The type of a structure is represented as the **ordered product of the types of its constituent fields**. For the terms $e$  and  $e'$ the rule for introducing a value of the product type$(\times)$  is:

$$\frac{\Gamma \vdash e : \mathbb{X} \ \Gamma \vdash e' : \mathbb{Y}}{\Gamma \vdash \langle e, e' \rangle : \mathbb{X \times Y}} \times I $$

#### Unions
> A union defines a type that can hold a value of one predefined type or another, called its variants. In a traditional context, it essentially permits a structure to have multiple interpretations. 

C unions are untagged, as in, they do not track which variant a union value is. Haskell unions are tagged however, pattern matching allows one to uncover which variant a union is currently holding.

In type theory, a tagged union is formally defined as the **Sum Type**. 
It is logically equivalent to disjunction. The terms associated with this type include constructors to mark the injected terms.

$$\frac{\Gamma \vdash e : A}{\Gamma \vdash inl(e) : A+B}+I_{1}$$
Is called left introduction. If the term $e$ has the type $A$, we can conclude that the tagged term $inl(e)$ has the type $A+B$ 

$$\frac{\Gamma \vdash e : B}{\Gamma \vdash inr(e) : A + B}+I_{2}$$
Is called right introduction. If the term $e$ has the type $B$, we can conclude that the tagged term $inr(e)$ has the type $A+B$.

The elimination rule governs the safe usage of the value of a sum type $A+B$, handling the two possibilities, and ensuring that have the same result type. 

$$\frac{\Gamma \vdash e : A+B \ \ \ \Gamma, x : A \vdash e' : C \ \ \ \Gamma, y : B \vdash e'' : C}{\Gamma \vdash \text{case (} inl(x) \implies e' \vee inr(y) \implies e'') : C}+E$$
### Function Types
> A function type is a type that represents a function taking an input of one type and producing an output of another type, often denoted as $A \to B$, representing functions with arguments of type $A$ and results of type $B$.

$$\frac{\Gamma, x : A \vdash e : B}{\Gamma \vdash \lambda x.e : A \to B} \to I$$
Is the function introduction rule, defining how a function receives its type. If $x$ has type $A$, you can derive that some expression $e$ has type $B$, you can form a function from $A$ to $B$. 


$$\frac{\Gamma \vdash f : A \to B \ \ \ \Gamma \vdash a : A} {\Gamma \vdash f \ \ a : B} \to E$$ Validates function application and determines its result type, elimination rule. 

If $f = \lambda x.e$ then applying $f$ to $a$ means:
$$(\lambda x.e) a \to_{\beta} e[a/x]$$
The expression $e$ where $x$ has been replaced by $a$.
### Equivalence
Provides the unambiguous rule necessary to answer whether two different type declarations are equivalent or compatible.  May be defined by a formal judgement of the form $\Gamma \vdash A = B$ in non-trivial cases. 

Historically, two approaches have been used to define type equivalence:

1. **Name Equivalence**
	This policy asserts that two types are equivalent if and only if they share the exact same name. This honours the programmer's deliberate choice, assuming that if the programmer chooses different names, the system should treat them as distinct types, regardless of their internal structure. 
2. **Structural Equivalence**
	This policy asserts that two types are equivalent if and only if they they have the same internal structure, regardless of syntactic convention. Requires that types consist of the same set of fields, in the same order, with those fields all having equivalent types. If a type system leverages this, the compiler must encode the type's structure to facilitate equivalence testing during semantic elaboration. Equivalence relation on types can be then computed recursively. 
In practice, a mixture of structural and by-name equivalence is used in most languages. 
### Type inference
> Type inference rules are the formal logical rules that tell us how to derive the type of an expression. 

Each rule, as seen prior, maintains the form $$\frac{premises}{conclusion}\text{Rule Name}$$They define what it means to say $\Gamma \vdash e : \tau$. 

A compiler uses these inference rules to calculate the type of every expression in the program. For example, if $e_1$ has type $int$ and $e_2$ also has type $int$, w know that $e_1 + e_2$ must also have type $int$. We write this as an **inference rule**:
$$\frac{ \Gamma \vdash e_{1} : int \ \ \ \Gamma \vdash e_{2} : int}{\Gamma \vdash e_1+e_{2} : int}$$
We can produce more of these!

*T-Var*
$$\frac{x:\tau \in \Gamma}{\Gamma \vdash x : \tau}$$
>A variable's type is whatever the context declares it to be.

*Function abstraction:*
$$\frac{\Gamma, x : \tau_{1} \vdash e : \tau_{2}}{\Gamma \vdash (\lambda x.e) : \tau_{1} \to \tau_{2} }$$


*Function application*
$$\frac{\Gamma \vdash e_{1} : \tau_{1} \to \tau_{2} \ \ \ \Gamma \vdash e_{2} : \tau_{1}}{\Gamma \vdash e_{1} \ e_{2} : \tau_{2}}$$

For example, we can infer the type of $\lambda x.x$ 
We start with an unknown type for $x$, et us say $\alpha$.
Via $T-Var$ we can conclude that the type for $x$ is whatever the context says it is - $\Gamma, x : \alpha \vdash x : a$ 
Via Function Abstraction we can infer that $\lambda x.x : \forall a.a \to a$, via $\Gamma \vdash \lambda x.x : \alpha \to \alpha$ 

To perform type inference, first must calculate the types of the arguments, and then look for an inference rule that matches. 
### Type promotion
> Type promotion, formally referred to as **implicit conversion**, occurs when a language automatically converts one type into another compatible type.

When dealing with mixed type expressions that would otherwise fail, the compiler is at liberty to insert conversion code, typically converting one or both operands to a more general type. This is more common in weakly typed languages. 

This often involves coercing the less precise value to the form of the more precise/compatible type before executing the operation. The most common example of this is with numeric promotion. 

There are many possible ways to achieve type promotion, for example producing promotion rules in the type system for different types, which we can refer to as subtyping. 