# CMPT886-Paper-1 #
Presentation on the CSmith, a C compiler bug finder
_By Geoff Groos, Rafiq Dandoo, and Sanjeet ???_

--

### Finding Bugs is Hard ###

Why not generate tests?

- Tests are dumb -> lack of expressability
- Tests are too large -> not short and concise 
- Language is too complex -> difficult to generate code

--

### Enter CSmith ###
Use a Stochastic AST generation scheme

- Generate short but punchy source code

Run the compilers on that generated source

- Disagreement about the behaviour of the same source code between different compilers implies a bug

--

### Why C (and not C++?) ###

- Lots of safety-critical embedded systems still written in C
 - really?

- Reasonably contained language with many vaguely defined constructs
 - `unsigned char x = ((byte)1)<<7`

- Still a big progamming community
 - All of whom are insane not to use ADA or C but with a C++ compiler...

--

…pointer Safety, Null—Pointer dereferences are easy to avoid using dynamic checks. There is, on the other hand, no portable run-time method for detecting references to a function-scoped variable whose lifetime has ended. (Hacks involving the stack pointer are not robust under inlining.) Although there are obvious ways to structurally avoid this problem, such as using a type system to ensure that a pointer to a function-scoped variable never outlives the function, we judged this kind of strategy to be too restrictive. Instead, Csmith freely permits pointers to local variables to escape (e.g., into global variables) but uses a whole-program pointer analysis to ensure that such pointers are not dereferenced or used in comparisons once they become invalid. Csmith’s pointer analysis is flow sensitive, field sensitive, context sensitive, path insensitive, and array-element insensitive. A points-to fact is an explicit set of locations that may be referenced, and may include two special elements: the null pointer and the invalid (out- of-scope) pointer. Points-to sets containing a single element serve as must-alias facts unless the pointed-to object is an array element. Because **Csmith  does  not  generate  programs  that  use  the  heap**, assigning names to storage locations is trivial. Effect safety The C99 standard states that “[t]he order of evalua- tion of the function designator, the actual arguments, and subexpres- sions within the actual arguments is unspecified.” Also, undefined behavior occurs if “[b]etween two sequence points, an object is modified more than once, or is modified and the prior value is read other than to determine the value to be stored.” To avoid these problems, Csmith uses its pointer analysis to perform a conservative interprocedural analysis and determine the effect of every expression, statement, and function that it generates. An  effect  consists  of  two  sets:  locations  that  may  be  read  and locations that may be written. Csmith ensures that no location is both read and written, or written more than once, between any pair of sequence points. As a special case, in an assignment, a location can be read on the RHS and also written on the LHS…​
