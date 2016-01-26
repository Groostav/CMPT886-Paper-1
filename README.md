# CMPT886-Paper-1 #

A review of **Finding and Understanding Bugs in C Compilers**

_By Geoff Groos, Rafiq Dandoo, and Sanjeet Tripathy_

---

# Why do we need to find compiler bugs? #

- Lots of programmers still working in C.  
- They want to make changes to their source code, and they dont want to be bitten by a bug in their compiler
 
   > Most problems with GMP these days are due to problems not in GMP, but with the compiler used for compiling the GMP sources. 
   > -- _gmplib.org on "Reporting a bug in GMP"_

---

# Finding Bugs is Hard #

Why not test the compilers with _random_ C source code?

- Tests are too large -> not short and concise
- Language is too complex -> difficult to write a tool that generates code
- Cant judge results, how do we know if the compiled code is correct?

---

# Enter CSmith #
Use a stochastic reverse-AST generation scheme

TODO graphs... source to AST and back again.

---

# Enter CSmith (2) #

- Using this scheme we _should be able_ generate good tests
 - First goal: Every program must be well formed and have a single meaning
 - Second goal: Maximize expressiveness (use a large but concise set of language features)
- Started as a fork of RandProg
 - tweaked to target more general concepts than `volatile` tests. 

---

# Determining correctness #

Once a random program is generated and compiled, how can we use it to find a bug?

- compile the program with different compilers
- run the programs output by each compiler
- if they do not all produce the same output, the one that produced a different output from the other two is assumed to have a bug.
 - similarly, the same compiler at different optimization levels should produce the same functional program

---

# Saftey Checks for Randomly generated code

_They spend a great deal of time making sure you know their program generates correct code_

Things they needed to worry about:
 - type Safety: not really clear how they did address this just said "required most care"
 - Pointer Safety: Csmith uses pointer analysis to preform conservative interprocedural analysis and determine effect of every expression.
 - Integer Safety: Cmsith uses a family of wrapper functions for arithmetic operators whose (promoted) operands might overflow
 - Array Safety: generates index variables
 - Initilizer Safety: local ones are easy to deal with, gotos complicate this, Csmith forbids gotos from spanning Initilizer code.

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
  unsigned int l_75 = 1, l_77 = 1, l_264 = 0;
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

What does this do?
Whats really incredible is the LLVM compiler guys get _right on it_, and find the problem in _30 minutes_

---

# Some Sample code (2) #

>**GCC Bug #2: wrong transformation**
>In C, if an argument of type `unsigned char` is passed to a function with a parameter of type `int`, the values seen inside the function should range from `0` to `255`. We found a case in which a version of GCC inlined this kind of function call and then sign-extended the argument rather than zero-extending it, causing the function to see negative values of the parameter when the function was called with arguments in the range `128` to `255`

So In other words

```C
int main(void){
  unsigned char x = 200;
  foo(x);
}

void foo(int input){
  print(input);
}
```

would be a nice test case...

---

# Some Sample code (2) (cont) #

```C
static unsigned char g_2 = 1;
static int g_9;
static int *l_8 = &g_9;

static void func_12(int p_13)
{
  int * l_17 = &g_9;
  *l_17 &= 0 < p_13;
}

int main(void)
{
  unsigned char l_11 = 254;
  *l_8 |= g_2;
  l_11 |= *l_8;
  func_12(l_11);
  printf("%d\n", g_9);
  return 0;
} 
```

_much better than the LLVM bug_, still not very _concise_.

---

# Trends #

These are some of the trends of the various GCC and LLVM compilers as per the tests generated by CSmith:

TODO:IMG

---

# Trends (cont) #

And its good news! Generally the compilers are getting better...

TODO:IMG

 - the v4 track of GCC has decreased its overall bug count
 - the versions of LLVm since 2.4 have largely been better than their predecessor

---

# Trends (cont 2) #

TODO: IMG w/ downward slope

---

# Comp Cert; a formally verified C Compiler #

> The middle end bugs we found in all other compilers are absent. As of early 2011, the under-development version of CompCert is the only compiler we have tested for which CSmith cannot find wrong-code errors.

So the only formally verified compiler they tested they couldn't find a number of bugs for.

---

# Criticms #

No heap support, mentioned admist a wall of text

>...such pointers are not dereferenced or used in comparisons once they become invalid. Csmith’s pointer analysis is flow sensitive, field sensitive, context sensitive, path insensitive, and array-element insensitive. A points-to fact is an explicit set of locations that may be referenced, and may include two special elements: the null pointer and the invalid (out- of-scope) pointer. Points-to sets containing a single element serve as must-alias facts unless the pointed-to object is an array element. Because **Csmith  does  not  generate  programs  that  use  the  heap**, assigning names to storage locations is trivial. Effect safety The C99 standard states that “[t]he order of evalua- tion of the function designator, the actual arguments, and subexpres- sions within the actual arguments is unspecified.” Also, undefined behavior occurs if “[b]etween two sequence points, an object is modified more than once, or is modified and the prior value is read other than to determine the value to be stored.” To avoid these problems, Csmith uses its pointer analysis to perform a conservative interprocedural analysis and determine the effect of every expression...


# Criticisms (2) #

Csmith never commits to a code fragment unless it has been shown to be safe.

Loops and functions complicate this. solution is dataflow analysis!

```C
int i;
int *pi = &i;
while(someCondition){
  p = 3;
  //...
  *p = 0;
}
```

Options: be conservative with analysis: run the whole program's analysis, which isn't efficient
Solution: restrict analysis to local scope where functions and loops are involved.

---

# Citicisms (3) #

In [one of the GCC bugs](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=42952) the code submitted as abug appears to be reduced (manually?) to an SSCCE, but he makes no mention of this process.

```C

static int g_16[1];

static int *g_135 = &g_16[0];
static int *l_15 = &g_16[0];

static void foo (void)
{
  g_16[0] = 1;
  *g_135 = 0;
  *g_135 = *l_15;
  printf("%d\n", g_16[0]);
}

int main(void)
{
   foo();
   return 0;
} 
```

That is the code submitted in the bug, but I suspect the generated program was much larger, since his 'program size' metric is on the order of KB, so how did he reduce this? Did he use a tool?

---

# Citicism (4) #

The effect of CSmith on statement/function/branch coverage does not change, and that path coverage analysis would likely show it as having a bigger impact

_So why didnt he run the path coverage analysis?_

---

# Pro/con list #

**Pros:**

- largely automated process, generate whatever and pass it to three compilers, requires very little involvement.
- 

**Cons:**

- tests are not minimal or concise -> not easy to read.

---

# Discussion # 

- Are there better strategies for bug finding?
 - He claims '300 bugs for $1000', but if he had simply be auditing the code, would he have found more bugs? 

- Why didn't he use some kind of test minimizer?
