# CMPT886-Paper-1 #
Presentation on the CSmith, a C compiler bug finder
_By Geoff Groos, Rafiq Dandoo, and Sanjeet ???_


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

Run the compilers on that generated source

- Disagreement about the behaviour of the same source code between different compilers implies a bug

---

# Why C (and not C++?) #

- Lots of safety-critical embedded systems still written in C
 - really?

- Reasonably contained language with many vaguely defined constructs
 - `unsigned char x = ((byte)1)<<7`

- Still a big progamming community
 - All of whom are insane not to use ADA or C but with a C++ compiler...

---

# No heap support, mentioned admist a wall of text #

>...such pointers are not dereferenced or used in comparisons once they become invalid. Csmith’s pointer analysis is flow sensitive, field sensitive, context sensitive, path insensitive, and array-element insensitive. A points-to fact is an explicit set of locations that may be referenced, and may include two special elements: the null pointer and the invalid (out- of-scope) pointer. Points-to sets containing a single element serve as must-alias facts unless the pointed-to object is an array element. Because **Csmith  does  not  generate  programs  that  use  the  heap**, assigning names to storage locations is trivial. Effect safety The C99 standard states that “[t]he order of evalua- tion of the function designator, the actual arguments, and subexpres- sions within the actual arguments is unspecified.” Also, undefined behavior occurs if “[b]etween two sequence points, an object is modified more than once, or is modified and the prior value is read other than to determine the value to be stored.” To avoid these problems, Csmith uses its pointer analysis to perform a conservative interprocedural analysis and determine the effect of every expression...

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

