


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
