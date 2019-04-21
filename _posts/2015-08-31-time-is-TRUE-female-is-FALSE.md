---
layout: post
title: "Time is TRUE, Female is FALSE"
date: 2015-08-31
categories: [R, logicals, true, false]
comments: true
---

## Introduction
R has two reserved words denoting logical constants, namely `TRUE` and `FALSE`. These are case sensitive literals and you cannot create a variable or a function named `TRUE` or `FALSE`.

```r
> TRUE <- 1
Error in TRUE <- 1 : invalid (do_set) left-hand side to assignment
> TRUE <- function() print("hello")
Error in TRUE <- function() print("hello") :
  invalid (do_set) left-hand side to assignment
> FALSE <- 1
Error in FALSE <- 1 : invalid (do_set) left-hand side to assignment
> FALSE <- function() print("hello")
Error in FALSE <- function() print("hello") :
  invalid (do_set) left-hand side to assignment
```

R also has two global variables, namely, `T` and `F` whose initial values are set to `TRUE` and `FALSE` respectively<sup>[1](#seehelp)</sup>. This allows us to write the following kind of code:

```r
> rm(list = ls())
> if (T) print("Yes") else print("No")
[1] "Yes"
> if (F) print("Yes") else print("No")
[1] "No"
```

Since `T` and `F` are simply global variables, we can see their values by typing out thee variable names:

```r
> T
[1] TRUE
> F
[1] FALSE
```

We can also re-assign new values to these variables:

```r
> T <- 1:10
> F <- "female"
> T
 [1]  1  2  3  4  5  6  7  8  9 10
> F
[1] "female"
```

After we have reassigned to these variables<sup>[2](#removeTFbase)</sup>, we can also delete these variables using `rm()`:

```r
# After having reassigned to T and F
> rm(T)
> rm(F)
> T
[1] TRUE
> F
[1] FALSE
```

You can already imagine the ambiguity in using `T` and `F` instead of `TRUE` and `FALSE`. Since, we can easily redefine `T` and `F`, we run the risk of unexpected, silent mistakes in our code<sup>[3](#titleref)</sup>. This problem is further exacerbated by R's lexical scoping rules, which we discuss next.

## Lexical scoping rules in R

[Scoping](https://en.wikipedia.org/wiki/Scope_(computer_science)) is a set of rules that allow a programming language to infer the value of a variable from the variable name. There are two main types of scoping - **lexical** and **dynamic**.
R has support for both these kinds of scoping rules. We will only look at lexical scoping through examples in this post. I recommend reading Hadley Wickham's _Advanced R_ online section on [Functions](http://adv-r.had.co.nz/Functions.html) and [this](https://github.com/jtleek/modules/blob/master/02_RProgramming/Scoping/index.md) github page for details. Interested reader may also want to see [this post](https://xianblog.wordpress.com/2010/09/13/simply-start-over-and-build-something-better/) purportedly by [Ross Ihaka](https://en.wikipedia.org/wiki/Ross_Ihaka), one of the creators of R.

Let's look at a simple example in which we define a function and try to call it:

```r
# In a fresh R session
> f <- function() print(x)
> f
function() print(x)
> f()
Error in print(x) : object 'x' not found
```

Note that we do not need to define the variable `x` before we define the function `f`. However, we need to define a reachable variable `x` before we call the function `f`. By reachable, I mean, the function `f` should be able to reach the variable `x`. In other words, the variable `x` should be within the scope of the function `f`.

Let's first look at the case in which we define a variable `x` in such a way that it is not reachable by `f`.

```r
> g <- function() { x <- 1 }
> g()
> print(g())
[1] 1
> f()
Error in print(x) : object 'x' not found
```

In the above example, we did define `x` but inside the function `g()`. The function `f()` cannot reach the variable `x` and we get an error when we try to call `f()`. Now, let's define `x` in the global environment

```r
> x <- 10
> f()
[1] 10
```

Now, the variable `x` defined in the global environment is reachable by `f()` and we can successfully call `f()`. Note that we haven't passed `x` through the function `f()`. This is a common feature of a functional programming language such as R. Furthermore, we can change the behavior of `f()` without actually changing `f()` at all!

```r
> x <- 20
> f()
[1] 20
```

This means that functions in R are not totally isolated from the environment the function is defined in. In fact, the function and the environment the function is defined in are very tightly related (see closures in [Functional Programming](http://adv-r.had.co.nz/Functional-programming.html#functional-programming) if you want to know the details). This kind of scoping rule is not used in other high-level, numeric programming languages such as Octave<sup>[4](#octave-scoping)</sup>.

We can take this experiment further by calling `f()` form inside another function:

```r
> x
[1] 20
> h <- function() { x <- 30; f() }
> h()
[1] 20
```

Now, even though we defined `x` to equal `30` inside the function `h()`, calling `f()` from inside `h()` still prints `x` as `20` (which was defined in the global environment). This is lexical scoping. As a quick experiment for the curious minded, what would happen if we did not even define `x` in the global environment ? This example shows the result:

```r
# In a fresh R session
> f <- function() print(x)
> x
Error: object 'x' not found
> h <- function() { x <- 30; f() }
> f()
Error in print(x) : object 'x' not found
```

We get an error! Since `x` wasn't defined in the environment of `f()`, `x` is still not reachable by `f()`. So, the behavior of `f()` depends upon two things:

1. Definition of `f()`, and
2. Environment in which `f()` was defined


To demonstrate this fact, let's do another experiment. This time we define `f()` inside another function:

```r
> hh <- function() { x <- 30; f <- function() print(x); f();}
> hh()
[1] 30
> x <- 10
> x
[1] 10
> hh()
[1] 30
```

This time the global environment does not influence the behavior of `f()`. We defined `f()` inside `hh()` and that's the only environment that matters. To fully understand this concept, let's so one final experiment:

```r
# In a fresh R session
> hhh <- function() { f <- function() print(x); f();}
> hhh()
Error in print(x) : object 'x' not found
> x <- 10
> hhh()
[1] 10
```

This time we did not define `x` inside `hhh()`. Since in a fresh R session, there is no `x` defined, we get an error when we call `hhh()` the first time.
Then, we define `x` in the global environment (which is not the environment `f()` was defined in!). Now when we call `hhh()` (and in turn, `f()`), we get no error.

_What happened?_

There was no `x` defined inside `hhh()` which is the environment in which `f()` was defined. How then was R able to find the global variable `x` ? This is because, when R does not find the variable `x` inside `hhh()`, it keeps on looking one level up, until it finds a match (or it doesn't, in which case R throws an error).

This _recursive scoping rule_ has a **tremendous effect**. We were able to **modify** the behavior of a function `f()` that was defined within another function `hhh()` without modifying either `f()` or `hhh()`!

In fact, _we can modify the behavior of any function that uses a non-formal argument<sup>[5](#non-formal-argument)</sup> by modifying an appropriate parent environment of the function_.

Now that we understand the scoping rules in R, let's get back to the problem of `T` and `F` in R.

## The Problem with `T` and `F`

Thee obvious benefit of using `T` and `F` instead of the full literals `TRUE` and `FALSE` is the reduced number of keystrokes. Admittedly, the reduced number of keystrokes required (with Shift pressed or CapsLock on) becomes very attractive when using the [interactive mode](http://www.perfectlyrandom.org/2015/05/16/colon-operator-in-R/). I am guilty of doing this myself.

Use of `T` and `F` is very dangerous, especially in programming mode. An obviously evil thing to do is to simply bind `T` to `FALSE` and `F` to `TRUE` and see everything fail (see Example 1). The dangers of using `T` and `F` can be far more subtler (see Example 2).

### Example 1

Let's define a simple dummy function:

```r
# In a fresh R session
tf <- function() {
  print(T)
  if (T) {
    print("T")
  } else {
    print("not T")
  }
}
# Call tf()
> tf()
[1] TRUE
[1] "T"

```

This simple dummy function works as expected. Using the scoping rules of R, we can modify the behavior of the `tf()` function without changing `tf()` at all:

```r
> # Now, redefine T and call tf()
> T <- FALSE
> tf()
[1] FALSE
[1] "not T"
```

### Example 2

Let's look at this problem in a more real-life context where we have multiple files being `source`d.

<script src="https://gist.github.com/ankur-gupta/582bfba52054b9e8d9b3.js"></script>

You can download/clone these [example files as a GitHub Gist here](https://gist.github.com/ankur-gupta/582bfba52054b9e8d9b3) and play around with the code.

I first define a function `oddmean()` in the file `oddmean.R`. This function uses `T` and `F` as non-formal function arguments instead of the corresponding literals. We then source the file `oddmean.R` in various different files and see the effects. When `T` and `F` are bound to `TRUE` and `FALSE` correctly, we get the correct answer as shown in `correctanswer.R`. But, when we redefine `T` or `F`, we get incorrect answers as shown in `wronganswer1.R` and `wronganswer2.R`.

Let's look at the file `wronganswer2.R`. We do not define any variables in this file at all. Instead, we simply source some of the files we already created and know to work correctly.

1. We obviously want to source the function `oddmean()` in `oddmean.R`.
2. Then, we source `redefine.R` that redefines `T` and `F`. In real life, this could be accidental or this could be someone else's code that we wish to use.
3. Finally, we source the file `correctanswer.R`, which we have tested separately and we know that it gives the correct answer.

Now, even though we only sourced files that we knew to give the correct result in some circumstances, **we still get the wrong result, simply because we sourced some other file!**

What is even more concerning is that R did not throw an error! Due to the forgiving nature of R and the support for various kinds of indexing (logical, numeric, character; see [this previous post](http://www.perfectlyrandom.org/2015/06/16/never-trust-the-row-names-of-a-dataframe-in-R/)), we get the wrong result silently. To the user, the incorrect value of 2.6 is not that far off from the correct value of 5 and this might go undetected. In fact, we don't even know how often these silent errors occur. Maybe a large number of these errors go unnoticed!

## The Solution

The solution is simple and obvious: use `TRUE` and `FALSE` instead of `T` and `F`. Let's analyze this situation in terms of some common questions.

1. **Why won't we encounter the same problem with `TRUE` and `FALSE`?**

    We won't have the same problem with `TRUE` and `FALSE` because these are reserved keywords or literals. R does not allow us to (re-)define a variable or a function named `TRUE` or `FALSE`.

    Using `TRUE` and `FALSE` will make our code **tamper-proof** (for this particular kind of problem; this doesn't solve the issues of lexical scoping in general).


2. **Why don't we simply `rm()` the global variables `T` and `F` ?**

    We can't! See footnote<sup>[2](#removeTFbase)</sup>.


3. **Can I use `T` and `F` as logicals in interactive mode?**

    If you are sure that `T` and `F` are correctly defined then yes. This can be easily checked by quickly typing out `T` and `F` and ensuring the output matches `TRUE` and `FALSE`.


4. **Are the extra keystrokes worth it?**

    **Absolutely!** The extra keystrokes required to fully type out `TRUE` and `FALSE` buy me some peace of mind. Even if I am sure that I did not define `T` and `F` anywhere in my code, it is possible that some other package that I'm using or some one else's code or even my own code from a few years ago could have redefined these variables. The risk of incorrect results is too high to skimp on the keystrokes. And, any code that I write using `T` and `F` will forever be susceptible to this problem. _I'd rather go to bed feeling good about my code then having avoided carpal tunnel._


5. **Can I use `T` for time or `F` for female ?**

    In my opinion, **no**! Even if you decide to never use `T` and `F` as logicals, some other package or someone else who uses your code might.


## Footnotes
<a name="seehelp"><sup>1</sup></a> See `?T`.

```r
?T
...
‘TRUE’ and ‘FALSE’ are reserved words denoting logical constants
in the R language, whereas ‘T’ and ‘F’ are global variables whose
initial values set to these.  All four are ‘logical(1)’ vectors.
...
```

<a name="removeTFbase"><sup>2</sup></a> We cannot remove the global variables `T` and `F` if we haven't reassigned these variables.

```r
# In a fresh R session
> rm(T)
Warning message:
In rm(T) : object 'T' not found
> rm(F)
Warning message:
In rm(F) : object 'F' not found
> T
[1] TRUE
> F
[1] FALSE
```

The above happens because both `T` and `F` are defined in the `base` package of R:

```r
# In a fresh R session
> library(pryr)
> where("T")
<environment: base>
> where("F")
<environment: base>
```

It so happens that R does not allow us to remove variables from the base environment, as mentioned in the documentation of the `rm()` function:

```r
?rm
...
It is not allowed to remove variables from the base environment
and base namespace, nor from any environment which is locked (see
‘lockEnvironment’).
...
```

As a result, we are stuck with `T` and `F` as always being reachable global variables, whether we have defined them explicitly or not.

However, once we reassign to `T` or `F`, we essentially create new variables in the `R_GlobalEnv` with these names.

```r
> T <- 1:10
> library(pryr)
> where("T")
<environment: R_GlobalEnv>
> T
 [1]  1  2  3  4  5  6  7  8  9 10
> base::T
[1] TRUE
```

This does not mean that the original `base::T` (or `base::F`) variable is reassigned. The newly created `T` and `F` variables simply mask the original `T` and `F` variables in `base` package.

<a name="titleref"><sup>3</sup></a> The title of this post refers to the ambiguity of the variables `T` and `F`. The variable name `T` is usually used to denote time, temperature, or a t-distributed random variable. Similarly, `F` could be used to denote gender (female), force, or an F-distributed random variable. Due to a simple error in coding, we could end up silently considering time as `TRUE` and female as `FALSE`. This title is adapted from
from _Section 8.3.5_, [Burns, Patrick. _The R Inferno_, **2011**](http://www.burns-stat.com/pages/Tutor/R_inferno.pdf).

<a name="octave-scoping"><sup>4</sup></a>Scoping rules in [GNU Octave](https://www.gnu.org/software/octave/) are not the same as R. Let's try the same example but in Octave:

```matlab
octave:2> function [] = f()
> disp(x)
> end
octave:3> f()
error: 'x' undefined near line 2 column 6
error: evaluating argument list element number 1
error: called from:
error:   f at line 2, column 1
```

Until here, both Octave and R have the same behavior. We can still define a function in Octave without passing the variable `x` through the function `f()`.
And, as a result, we get the expected error indicating that `x` is not defined. Now, let's define `x` in the global environment and see if we can call `f()` successfully:

```matlab
octave:3> x = 10
x =  10
octave:4> f()
error: 'x' undefined near line 2 column 6
error: evaluating argument list element number 1
error: called from:
error:   f at line 2, column 1
octave:4> x
x =  10
```

This is where R and Octave differ. Even though `x` is defined in the global namespace, we cannot reach `x` from `f()` without explicitly passing `x` through `x` as a formal function argument.

```matlab
octave:5> function [] = f(x)
> disp(x)
> end
octave:6> f(x)
 10
```

This means, in Octave, unlike R, a function is completely isolated from the environment it was defined in (for the most part, things do get complicated in some advanced cases).

<a name="non-formal-argument"><sup>5</sup></a>A formal argument of a function is the variable passed through the function signature. In the following example, `y` is a formal argument of the function `mixf()` but `x` is non-formal argument.

```r
> mixf <- function(y) { print(x); print(y) }
> x <- 10
> mixf(20)
[1] 10
[1] 20
```

Some issues with scoping may be avoided by using only the formal arguments. However, since R is [homoiconic](https://en.wikipedia.org/wiki/Homoiconicity) and [functional](https://en.wikipedia.org/wiki/Functional_programming), enforcing formal arguments is not as satisfying as you'd think.

Furthermore, if you enforce the use of formal arguments (say, by defining a function inside an empty environment), this will make the function unusable (or extremely tedious) in practice. See the **Dynamic lookup** section [here](http://adv-r.had.co.nz/Functions.html) for an example.

This is because in R:

1. Even the basic operators such as ["+" are functions](http://www.perfectlyrandom.org/2015/05/16/colon-operator-in-R/#operator) defined in the base package's environment, and

2. R uses lexical scoping to find these functions as well (just like it does for variables).


