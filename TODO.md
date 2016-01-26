


If geoffs graph content goes all wrong:

# Algorithm for Generating Random Code

  - Starts by creating type definitions

  - Begins to generate main()
  - checks what it's allowed to produce from it's grammar, using a probability table and filter
  - if the selected production requires a target, eg variable or function it selects or defines one
  - generator uses recursion if the selected production is nonterminal
  - executes a collection of dataflow transfer functions
  - does saftey checks (more on this)

---


where to put htis?

- appendix containing a complete list of bugs?

- Why C and not C++/PHP/JS/Java/C#?
 - Lots of safety-critical embedded systems still written in C
  - really?
  
 - Lots of bugs in boxing/unboxing of integer-ish types in C
  - What does this do? Its _probably_ well defined...
   - `unsigned char x = (char)((byte)1)<<7`
    
- Still a big progamming community
 - All of whom are insane not to use ADA or C but with a C++ compiler...

- C++ is extremely complex and creating a random generator for it would be much harder
 - other languages are frequently more complex
