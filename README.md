# CMPT886-Paper-1 #
Presentation on the CSmith, a C compiler bug finder
_By Geoff Groos, Rafiq Dandoo, and Sanjeet ???_


## 1: intro ##

---

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

## 4: criticisms ##

--- 

### No heap support, mentioned admist a wall of text ###

>...pointer Safety, Null—Pointer dereferences are easy to avoid using dynamic checks. There is, on the other hand, no portable run-time method for detecting references to a function-scoped variable whose lifetime has ended. (Hacks involving the stack pointer are not robust under inlining.) Although there are obvious ways to structurally avoid this problem, such as using a type system to ensure that a pointer to a function-scoped variable never outlives the function, we judged this kind of strategy to be too restrictive. Instead, Csmith freely permits pointers to local variables to escape (e.g., into global variables) but uses a whole-program pointer analysis to ensure that such pointers are not dereferenced or used in comparisons once they become invalid. Csmith’s pointer analysis is flow sensitive, field sensitive, context sensitive, path insensitive, and array-element insensitive. A points-to fact is an explicit set of locations that may be referenced, and may include two special elements: the null pointer and the invalid (out- of-scope) pointer. Points-to sets containing a single element serve as must-alias facts unless the pointed-to object is an array element. Because **Csmith  does  not  generate  programs  that  use  the  heap**, assigning names to storage locations is trivial. Effect safety The C99 standard states that “[t]he order of evalua- tion of the function designator, the actual arguments, and subexpres- sions within the actual arguments is unspecified.” Also, undefined behavior occurs if “[b]etween two sequence points, an object is modified more than once, or is modified and the prior value is read other than to determine the value to be stored.” To avoid these problems, Csmith uses its pointer analysis to perform a conservative interprocedural analysis and determine the effect of every expression, statement, and function that it generates. An  effect  consists  of  two  sets:  locations  that  may  be  read  and locations that may be written. Csmith ensures that no location is both read and written, or written more than once, between any pair of sequence points. As a special case, in an assignment, a location can be read on the RHS and also written on the LHS…​

--

### Some sample code ###

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

The code that CSmith generted to produce this bug report was _this_:

```C
#include <stdio.h>

static inline unsigned long int
mod_rhs(const long int rhs)
{
    if (rhs == 0) return 1;
    return rhs;
}

static inline unsigned long int
div_rhs(const long int rhs)
{
    if (rhs == 0) return 1;
    return rhs;
}

static inline unsigned int
rshift_u_u(unsigned int left, unsigned int right)
{
	if (right >= sizeof(unsigned int)*8) {
		return left;
	}
	return left >> right;
}

unsigned int g_52;
unsigned int g_241;
unsigned int g_247;
unsigned int g_253;

int func_8 (unsigned int p_10);
int func_8 (unsigned int p_10)
{
  unsigned int l_11 = 0;
  for (1; l_11; l_11 -= 1)
    {
    }
  return 1;
}

int func_65 (unsigned short p_67, int p_69);
int func_65 (unsigned short p_67, int p_69)
{
  unsigned int l_70 = 0xD9F5B60AL;
  return l_70;
}

int func_76 (signed char p_78);
int func_76 (signed char p_78)
{
  p_78 = 1 % mod_rhs (1);
  return 1;
}

int func_73 (int p_74);
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

int main (void)
{
  func_73 (0);
  printf ("%d\n", g_253);
  return 0;
}
```

raise your hand if you can tell me what that does?

Whats really incredible is the LLVM compiler guys get _right on it_, and find the problem in _30 minutes_

--

### Nit-picking ###

- appendix containing a complete list of bugs?

