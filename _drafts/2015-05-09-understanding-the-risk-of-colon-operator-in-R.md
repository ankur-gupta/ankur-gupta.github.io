---
layout: post
title: Understanding the risk of colon operator in R 
date: 2015-05-09
categories: [R, colon]
---

### Background
Colon (`":"`) is an _operator_<sup>[1](#operator)</sup> in R<sup>[2](#version)</sup> that generates regular sequences. We can take a look at the documentation of the `":"` _function_<sup>[1](#operator)</sup> by typing in

```r
> ?":"
```
at the prompt.

We can generate a `vector` of integers from 1 to 10 (both inclusive), in increasing order, by using `":"`

```r
> 1:10
 [1]  1  2  3  4  5  6  7  8  9 10
```

This colon operator is a double-edged sword in the sense that we can also
generate a `vector` of integers from 10 to 1 in decreasing order using the exact same operator

```r
> 10:1
 [1] 10  9  8  7  6  5  4  3  2  1
```

Whether the sequence is increasing or decreasing, depends on whether the first argument (the number before the : sign) is smaller or larger than the second 
argument (the number after the : sign). Let's see some more examples to get familiar with this function.

```r
> 1:1
[1] 1
> 1:1.1
[1] 1
> 1:-1
[1]  1  0 -1
```

As long as both arguments to the `":"` function are `numeric` (or `integer`), we will always get back a `vector` of a length of at least one. This behavior may sound reasonable when we interact with the R prompt but this behavior may cause problems while writing scripts if we're not careful. 

<br/>

### The Risk
The use of colon operator is risky only if resulting behavior is unexpected. 
The **common mistake** is to assume that `x:y` will **always return an increasing sequence** from `x` to `y`.

For those switching from Octave/MATLAB to R, this is an especially common mistake. In Octave 3.8.2, `x:y` always returns an increasing 
sequence from `x` to `y`. This means, when `x > y`, the returned sequence is an empty vector, as shown in this example

```octave
octave:1> 10:1
ans = [](1x0)
```
Most experienced Octave/MATLAB users tend to transfer their knowledge via a 
_MATLAB to R conversion guides_ (such as [R for MATLAB users](http://mathesaurus.sourceforge.net/octave-r.html) and [MATLAB/R Reference](http://www.math.umaine.edu/~hiebeler/comp/matlabR.pdf)). These guides often do not emphasize the subtle difference between the `:` constructs in these languages. 


Coming back to R, the colon operator is most often used in two situations.

1. Indexing
2. `for` loops

Let us look at both of these cases through examples. 

##### Indexing


##### `for` loops

Colon operator is often used with the `length()` function to execute for loops over the length of a vector. For example, consider the following extremely trivial function that accepts a `list` of `vector`s and computes the deviation from mean

```r
compute_mean <- function(x) {
    for (i in 1:length(x)) {
        # In a real-life use case, these will be a complicated operation here.
        x[i] <- x[i] / 10
    }
    return(x)
}
```

The most use of colon operator is in indexing. 




Ankur Gupta Good for interactive sessions 
(1) where you get instant feedback
(2) where you can redo the computation after getting the instant feedback
(3) where you use hardcoded numbers instead of variables

Example:
1:10 
10:1 

Great, no chance of ambiguity or a mistake. 

Think of the absolute difference operation. You want to find out the absolute difference between 1 and 10. 
You can try 1 - 10, it'll give you wrong results. You can then try 10 - 1 and you'll get the correct results. There are only 2 "reasonable" options. You won't do 1 * 10 to get the absolute difference. 

Let's take x and y instead of 1 and 10. Now, again, you can do x- y first see if it's positive. If not, you can try y - x.
But if you want to get the absolute difference without instant feedback, ability to correct and hardcoded numbers, then you want to use the abs(x-y) construct. 

Can you do the same thing with colon? I want a set of increasing numbers from x to y, increasing by 1. You cannot use colon. You need to use seq(x, y, by = 1). 

So, what's the problem? The problem is very small. It's about fast typing and consistency. 

If you're doing an interactive session, typing seq(x, y, by = 1) is slow and overkill. You print x and y. You see if x <=y, then you do x:y; If not, you know the answer is c(). Easy!

If you're writing code/script/programming, now you can either (1) type in the above logic using if (which will get shot down during code review). Plus, this is tedious too. This is much like programming in a low level language. Or, (2) you can remember to not use the colon operator. This means, there are now two dialects of the R language you need to learn. -- one is the less formal, interactive dialect and the other is the formal, programming dialect. This is very much like English. You need to learn both written and spoken English. And you need to learn the correspondence between the interactive and programming dialects. 

I used to a language in which you don't need to this (or you need to do this very infrequently). 




Yesterday at 11:40am
Ankur Gupta NOTE! 
seq(10, 1, by = -1) works 
seq(1, 10, by = 1) fails with an error. 

You need to use an if


<br/><br/><br/>

#### Footnotes

<a name="operator"><sup>1</sup></a> There are no real operators in R. All operators are functions, even `"["`. See [this post by Hadley Wickham](http://adv-r.had.co.nz/Functions.html#all-calls). Try `get(":")` in R to see how the colon (":") function is defined. As a fun aside, we can invoke `":"` in a more common manner as follows

```r
> ":"(1, 10) # Common function form
 [1]  1  2  3  4  5  6  7  8  9 10
> base::":"(1, 10) # Useful when ":" gets redefined accidentally
 [1]  1  2  3  4  5  6  7  8  9 10
> .Primitive(":")(1, 10) # Useful when ":" gets redefined accidentally
 [1]  1  2  3  4  5  6  7  8  9 10
> 1:10 # Operator form
 [1]  1  2  3  4  5  6  7  8  9 10
```

Since `":"` happens to be a primitive function, there is really no R code of the `":"` function to look at. We can, however, take a look at the C definition of the `":"` function/operator. As discussed in [this Stack Overflow post](http://stackoverflow.com/questions/14035506/how-to-see-the-source-code-of-r-internal-or-primitive-function), it will help to first take a look at the `R-3.2.0/src/main/names.c` file. This file contains the following on Line 184

```c
{":",       do_colon,   0,  1,  2,  {PP_BINARY2, PREC_COLON,  0}},
```
This shows that we should be looking for the `do_colon` C function in the C source of R. Doing a simple **find/xargs/grep** magic, we can see that `R-3.2.0/src/main/seq.c` contains the definition of the `do_colon` C function. 

```
ankur@hogwarts ~/D/R/s/main> find . -name "*.c" -type f | xargs grep --color=auto "do_colon"

./eval.c:   Builtin2(do_colon, R_ColonSymbol, rho);             \
./names.c:{":",     do_colon,   0,  1,  2,  {PP_BINARY2, PREC_COLON,  0}},
./seq.c:/* The x:y  primitive calls do_colon(); do_colon() calls cross_colon() if
./seq.c:SEXP attribute_hidden do_colon(SEXP call, SEXP op, SEXP args, SEXP rho)
```

Trudging through the `do_colon` function, we reach the `seq_colon` function, which implements the `":"` functionality for integers and real numbers. 


<a name="version"><sup>2</sup></a> Version of R used in this post. 

```r
> version
               _
platform       x86_64-apple-darwin13.4.0
arch           x86_64
os             darwin13.4.0
system         x86_64, darwin13.4.0
status
major          3
minor          2.0
year           2015
month          04
day            16
svn rev        68180
language       R
version.string R version 3.2.0 (2015-04-16)
nickname       Full of Ingredients
```
