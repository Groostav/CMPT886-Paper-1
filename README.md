# CMPT886-Paper-1 #
Presentation on the CSmith, a C compiler bug finder
By Geoff Groos, Rafiq Dandoo, and Sanjeet Tripathy


---

# Finding Bugs is Hard #

Why not generate tests?

- Tests are dumb -> lack of expressability
- Tests are too large -> not short and concise
- Language is too complex -> difficult to generate code

---

# Enter CSmith #
Use a Stochastic AST generation scheme

- Generate short but punchy source code

- Started as a branch off another random C generator

---

# Design Goals of CSmith

Two main design goals of CSmith

  - First: Every program must be well formed and have a single meaning

  - Second: Maximize expressiveness with respect to restraints imposed by
    the first goal

---

# Methodology of Research

How do we tell if the compiled version of randomly generated source code is
correct when the source code is potentially thousands of lines long

  - We need an oracle of some type
    Problem is, as discussed in class this is a very hard thing to do

  - Better solution: poll the results of different compilers and ones that
    disagree with the majority are wrong

---

# Why C (and not C++?) #

- Lots of safety-critical embedded systems still written in C
 - really?

- Reasonably contained language with many vaguely defined constructs
 - `unsigned char x = ((byte)1)<<7`

- Still a big progamming community
 - All of whom are insane not to use ADA or C but with a C++ compiler...

- C++ is extremely complex and creating a random generator for it would be much harder

---

# No heap support, mentioned admist a wall of text #

>...such pointers are not dereferenced or used in comparisons once they become invalid. Csmith’s pointer analysis is flow sensitive, field sensitive, context sensitive, path insensitive, and array-element insensitive. A points-to fact is an explicit set of locations that may be referenced, and may include two special elements: the null pointer and the invalid (out- of-scope) pointer. Points-to sets containing a single element serve as must-alias facts unless the pointed-to object is an array element. Because **Csmith  does  not  generate  programs  that  use  the  heap**, assigning names to storage locations is trivial. Effect safety The C99 standard states that “[t]he order of evalua- tion of the function designator, the actual arguments, and subexpres- sions within the actual arguments is unspecified.” Also, undefined behavior occurs if “[b]etween two sequence points, an object is modified more than once, or is modified and the prior value is read other than to determine the value to be stored.” To avoid these problems, Csmith uses its pointer analysis to perform a conservative interprocedural analysis and determine the effect of every expression...

---

# Algorithm for Generating Random Code

  - Starts by creating type definitions

  - Begins to generate main()
  - checks what it's allowed to produce from it's grammar, using a probability table and filter
  - if the selected production requires a target, eg variable or function it selects or defines one
  - generator uses recursion if the selected production is nonterminal
  - executes a collection of dataflow transfer functions
  - does saftey checks (more on this)

---

# Saftey Checks for Randomly generated code

  Things they needed to worry about:
    - Integer Safety: Cmsith uses a family of wrapper functions for arithmetic operators whose (promoted) operands might overflow
    - type Safety: not really clear how they did address this just said "required most care"

    - Pointer Safety: Csmith uses pointer analysis to preform conservative interprocedural analysis and determine effect of every expression.

    - Array Safety: generates index variables

    - Initilizer Safety: local ones are easy to deal with, gotos complicate this, Csmith forbids gotos from spanning Initilizer code.

---

# Effcient Global Saftey

  Csmith never commits to a code fragment unless it has been shown to be safe.

  Loops and functions complicate this. solution is dataflow analysis!

  int i;
  int *pi = &i;
  while(...){
    p = 3;
    ...
    *p = 0;
  }
  **
  Options: be conservative with analysis: run the whole program's analysis
      - not Effcient
  Solution: restrict analysis to local scope where functions and loops are involved.

---

# Some sample code #

>**LLVM Bug #1: wrong safety check**
>`(x==c1)||(x<c2)` can be simplified to `x<c2` when `c1` and `c2` are constants and `c1 < c2`. An LLVM version incorrectly transformed `(x==0)||(x<-3)` to `x<-3`. LLVM did a comparison between `0` and `-3` in the safety check for this optimization, but performed an unsigned comparison rather than a signed one, leading to it to incorrectly determine that the transformation was safe.

You would think that the source code might look something like:

```C
int main(){
   int x = func1();
   int y = func2();

   if(x == 0) || (x < -3){
     x = func3();
   }

   y = func4();

   print(x, y);
}
```

_Not even close_.

---

# Some sample code (cont) #

The code that CSmith generted to produce this bug report was _this_:

```C
int func_73 (int p_74)
{
  unsigned int l_75 = 1;
  unsigned int l_77 = 1;
  unsigned int l_264 = 0;
  func_65 (((rshift_u_u
	     (l_75,
	      (func_76 (p_74)
	       || l_75 + func_8 (1)) <= 1 & 1 < (g_241 ^ (1 ==
							  p_74)) *
	      (func_65 ((g_247 * l_77), 1)
	       && (g_241 & (1 >= l_77))))) - 1), p_74 && ((g_52 | l_77)
							  && l_75
							  && (1 /
							      div_rhs (1))) *
	   ((p_74 ^ (func_8 (1) != (1 % mod_rhs (1)))) <= 1) - (1 >= l_264));
  g_253 = (p_74 && (p_74 > func_65 (0x76L, 1)));
  return 1;
}
```

raise your hand if you can tell me what that does?

Whats really incredible is the LLVM compiler guys get _right on it_, and find the problem in _30 minutes_

---

# Nit-picking #

- appendix containing a complete list of bugs?
