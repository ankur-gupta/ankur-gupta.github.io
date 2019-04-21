---
layout: post
title: The Colon "Operator" in R
date: 2015-05-16
categories: [R, colon]
comments: true
---

## Background
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

## The Risk
The use of colon operator is risky only if resulting behavior is unexpected.
A **common mistake** is to assume that `x:y` will **always return an increasing sequence** from `x` to `y`.

For those switching from Octave/MATLAB to R, this is an especially common mistake. In Octave 3.8.2, `x:y` always returns an increasing
sequence from `x` to `y`. This means, when `x > y`, the returned sequence is an empty vector, as shown in this example

```octave
octave:1> 10:1
ans = [](1x0)
```
Most experienced Octave/MATLAB users tend to transfer their knowledge via a
_MATLAB to R conversion guide_ (such as [R for MATLAB users](http://mathesaurus.sourceforge.net/octave-r.html) and [MATLAB/R Reference](http://www.math.umaine.edu/~hiebeler/comp/matlabR.pdf)). These guides often do not emphasize the subtle difference between the `:` constructs in these languages.


Coming back to R, the colon operator is most often used in two situations.
1. **`for` loops**
2. **Indexing**

Let us look at both of these cases through examples.

#### `for` loops
Colon operator is often used with the `length()` function<sup>[3](#forloop)</sup> to execute for loops over the length of a vector. For example, consider the following extremely trivial function that accepts a `list` of plottable objects.

```r
plot.objects <- function(x) {
    for (i in 1:length(x)) {
        print(i)
        plot(x[[i]])
    }
}
```

When the input `list` to the `plot.objects()` function is non-empty, everything works out nicely.

```r
x1 <- list(cars[1:5, ], diamonds[1:5, ])
plot.objects(x1)

# Output
# > x1 <- list(cars[1:5, ], diamonds[1:5, ])
# > plot.objects(x1)
# [1] 1
# [1] 2
```

But, when the input `list` is empty, `plot.objects()` function throws an error.

```r
x2 <- list()
plot.objects(x2)

# Output
# > x2 <- list()
# > plot.objects(x2)
# [1] 1
# Error in x[[i]] : subscript out of bounds
```

Since `length(x2)` is `0`, `1:length(x2)` is not empty, instead it is a vector of two elements.

```r
> length(x2)
[1] 0
> 1:length(x2)
[1] 1 0
```

The body of the `for` loop is executed for `i = 1` which fails because `x2[[1]]` does not exist. Luckily, in this case, attempting to access an index outside bounds throws an error. However, this is not always the case in R as we can see below

```r
> list()[[1]]
Error in list()[[1]] : subscript out of bounds
> list()[1]
[[1]]
NULL

> c()[1]
NULL
```

In general, R allows us to access any element of a vector<sup>[4](#vector_is_list)</sup> without giving an error, as evident from the examples below.

```r
> c()[0]
NULL
> c()[-1]
NULL
> x <- 1:10
> x
 [1]  1  2  3  4  5  6  7  8  9 10
> x[-1]
[1]  2  3  4  5  6  7  8  9 10
> x[0]
integer(0)
> x[-10]
[1] 1 2 3 4 5 6 7 8 9
> x[-100]
 [1]  1  2  3  4  5  6  7  8  9 10
> x[100]
[1] NA
>
```

In any programming language, absence of error is not sufficient to prove the correctness of code. This is especially important while coding in R.
Given R's very forgiving nature, we need to be extra careful to ensure correctness.

Let's look at another example where use of the color operator produces unintended effects. The following function computes the sum over a vector.

```r
mysum <- function(x) {
    sumval <- 0
    for(i in 1:length(x)) {
        sumval <- sumval + x[i]
    }
    return(sumval)
}
```

We would never really use `mysum()` function. R already has an in-built `sum()`function which is much more efficient. We will use `mysum()` to demonstrate why the use of the `1:length(x)` construct is dangerous.

Luckily, everything works when the input `vector` to the `mysum()` is not empty

```r
> x <- c(1, 2, 3)
> mysum(x)
[1] 6
> sum(x)
[1] 6
> identical(mysum(x), sum(x))
[1] TRUE
```

But, when the input `vector` is empty, `mysum()` produces unintended effects.

```r
> x <- c()
> mysum(x)
numeric(0)
> sum(x)
[1] 0
> identical(mysum(x), sum(x))
[1] FALSE
```

Notice how `mysum()` did not fail this time even when we accessed the first element of an empty vector. Even though `mysum()` does not throw an error, the result may be unexpected.

The question is, how do we define the sum of an empty vector? R defines it as `sum(c()) = 0`, which seems reasonable. The function `mysum()` actually agrees with this definition because it assigns the value `0` to the variable `sumval`before using the input vector `x`. It's the `for` loop in `mysum()` that causes the following incorrect result to be returned.

```r
> 0 + c()[1] + c()[0]
numeric(0)
```

Had the `for` loop not executed when `x = c()`, the desired value of `0` would have been returned.
So, how do we prevent `for` loop execution for an empty vector? The answer is simple, as
demonstrated by this snippet

```r
for(i in c()) {
    print(i)
}
# Output
# Prints nothing
```

A `for` loop over an empty `vector` is not executed. So, the problem is not with the `for` loop,
it is with the colon operator-based construct `1:length(c())`. Yet another common source
of bugs is the incorrect use of following constructs

```r
x <- c(1, 2, 3, 4)

# Error-prone constructs
# > 1:length(x) + 1
# [1] 2 3 4 5
# > 1:length(x) - 1
# [1] 0 1 2 3
# > 1:(length(x) + 1)
# [1] 1 2 3 4 5
# > 1:(length(x) - 1)
# [1] 1 2 3
```

Many users confuse `1:length(x) + 1` for `1:(length(x) + 1)` and vice-versa.

#### Indexing
The colon operator is used extensively in indexing and slicing arrays. For example, the following function computes the mean of first `n` elements of a vector.

```r
mean.first.n <- function(n, x) {
    mean(x[1:n], na.rm = TRUE)
}
```

Let's look at a few examples.

```r
x <- c(3, 2, 9, 1)

# Examples
# > mean.first.n(2, x)
# [1] 2.5
# > mean.first.n(1, x)
# [1] 3
# > mean.first.n(0, x)
# [1] 3
```

When `n` is greater than `0`, the function works correctly. When `n = 0`, we get the same result as we get for `n = 1`. This is again because of the colon operator.

We can ask the question, how do we define the mean of a 0-length vector? R defines this as `mean(numeric(0)) = NaN`. However, the indexing performed using the colon operator causes the following computation to be returned

```r
> mean(c(3, 2, 9, 1)[1:0])
[1] 3
```

The construct `mean(x[1:0])` will always return the first element of `x` (assuming `x` is non-empty). This is definitely ambiguous, if not outrightly incorrect.

<br/>
### Solution
The above examples demonstrate that the colon (`":"`) operator requires careful thought before use. One obvious solution is to separate the case where the colon operator returns a decreasing sequence of numbers. For example, the
`mysum()` function may be re-written as

```r
mysum2 <- function(x) {
    sumval <- 0
    if (length(x) > 0) {
        for(i in 1:length(x)) {
            sumval <- sumval + x[i]
        }
    }
    return(sumval)
}

# Output
# > x <- c()
# > mysum2(x)
# [1] 0
# > sum(x)
# [1] 0
```

Wrapping the `for` loop within an `if` solves the problem. However, there is a better solution. As we discussed, `for` loop is smart enough to handle empty vectors but the `:` operator does not return an empty vector.

There happens to be a function called `seq_along()` that is perfectly suited to this situation.

```r
x <- c(3, 2, 9, 1)
# > seq_along(x)
# [1] 1 2 3 4

x <- c()
> seq_along(x)
integer(0)
```

`seq_along()` generates a sequence of indices over a `vector` correctly. When a `vector` is empty, `seq_along()` returns an empty sequence.

```r
mysum3 <- function(x) {
    sumval <- 0
    for(i in seq_along(x)) {
        sumval <- sumval + x[i]
    }
    return(sumval)
}

# Output
# > x <- c()
# > mysum3(x)
# [1] 0
# > sum(x)
# [1] 0
```

Using `seq_along()` obviates the need for an `if` and generates the correct solution.

Similarly, when we want to obtain the first `n`-elements of a vector, we can use `seq_len(n)` instead of `1:n`.

```r
> 1:0
[1] 1 0
> seq_len(0)
integer(0)
```

We can now correct the function to compute the mean of first `n` elements.

```r
mean.first.n2 <- function(n, x) {
    mean(x[seq_len(n)], na.rm = TRUE)
}
x <- c(3, 2, 9, 1)

# Examples
# > mean.first.n2(2, x)
# [1] 2.5
# > mean.first.n2(1, x)
# [1] 3
# > mean.first.n2(0, x)
# [1] NaN
```

## Discussion
R has two modes of use
1. **Interactive Mode**
2. **Programming Mode**

The interactive mode is the use of interpreter (or command line) while the programming mode is the use of R scripts. Interactive mode allows the following advantages over the programming mode
1. We get instant feedback
2. We can redo the computation based on feedback
3. More use of hardcoded numbers instead of variables

The colon (`:`) operator is more suited to the interactive mode. Because we get instant feedback, it is easy to verify that any R code does exactly what we want it to do. Even if we make a mistake, interactive mode allows us to correct our mistake immediately by re-assigning the variable. Further, we often use a lot more hardcoded numbers (such as `x[1:10]`) instead of variables (such as `x[1:n]`), which reduces the potential for error.

Programming mode involves writing scripts and then executing/sourcing scripts as a batch. This is much more formal than the interactive mode. This difference is much like spoken and written modes of English. While speaking to an audience, say in a technical presentation, we get instant feedback (both verbal and non-verbal such as  facial expressions) and we can correct a mistake immediately. In written form of communication, say a research article, we do not get immediate feedback or the ability to correct ourselves.

The functions `seq_along()` and `seq_len()` are more suited to the programming mode. Obviously, the use of `:` involves fewer keystrokes compared to `seq_len()`, which is one of the reasons people prefer it for interactive use.

Even in programming mode, many R users experiment with code snippets in the interactive mode. They try out many different functions, constructs and approaches. Once they are satisfied with their code snippet, they transfer their perfected code snipped to their R script. The problem with this approach is that the code snippet that works correctly in the interactive mode for a particular example may not work in the formal (and more general) setting of the programming mode.

The `:` operator is a perfect example of this. The `:` operator needs to be converted into `seq_along()`, `seq_len()` or something entirely different to be robust. This creates an additional cognitive burden for the programmer. After we've spent a lot of time perfecting a code snippet in interactive mode, we need to start further modify the code snippet to make it work in general.

An obvious solution is to always use the formal, programming mode constructs (such as `seq_along()` and `seq_len()`), even on the R command line. This would be great except that this means typing too many characters on the R command line. The command line is not a text editor. It does not allow us to copy/paste text at will like Emacs, vim, Sublime Text, or even Notepad. Typing long lines at the command line is a tedious process (even with the tab completion).

R users routinely contribute R packages. Even if we abstain from the use of `:` operator in our own code, we will come across a lot of other people's code that uses `:` all the time. Thus, understanding how `:` works is essential to being a better R programmer.

So, how do we use the `:` operator? Instead of completely forbidding the use of `:` operator, in the next section, we define some rules that will help us avoid all mistakes.

## Summary
As we discussed, a complete understanding of the `:` operator is essential, especially the fact that `1:0 = c(1, 0)`. In addition, the following one rule will help us avoid all mistakes.

<blockquote>
<p>
  Use `:` only with hardcoded numbers, both in programming mode and interactive mode. For example, use of `1:10` in R scripts is okay.
  Use of `1:n` is dangerous and should be avoided.
</p>
</blockquote>

The following table is a quick summary of what we discussed.

| Instead of        |            | Use               |
|---------------    |  :-----:   |----------------   |
| `1:length(x)`     |   &rarr;   | `seq_along(x)`    |
| `1:n`             |   &rarr;   | `seq_len(n)`      |


## Footnotes
<a name="operator"><sup>1</sup></a> There are no real operators in R. All operators are functions, even `"["`. See [this post by Hadley Wickham](http://adv-r.had.co.nz/Functions.html#all-calls). Try `get(":")` in R to see how the colon (`":"`) function is defined. As a fun exercise, we can invoke `":"` in a more common manner as follows

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

<a name="forloop"><sup>3</sup></a> See _Section 8.1.60_, [Burns, Patrick. _The R Inferno_, **2011**](http://www.burns-stat.com/pages/Tutor/R_inferno.pdf).

<a name="vector_is_list"><sup>4</sup></a> A `list` is also a `vector`. We can create a list using the `vector()` function

```r
> x <- vector("list", 2)
> x
[[1]]
NULL

[[2]]
NULL

> class(x)
[1] "list"
```

We can perform many operations on `list`s like on `vector`s. For example, we can combine `list`s just like we combine `vector`s

```r
> x <- list(1, 2)
> y <- list(3, 4)
> z <- c(x, y)
> class(z)
[1] "list"
> z
[[1]]
[1] 1

[[2]]
[1] 2

[[3]]
[1] 3

[[4]]
[1] 4
```

We can also slice a `list` like a `vector`

```r
> z <- list(1, 2, 3, 4)
> z[2:3]
[[1]]
[1] 2

[[2]]
[1] 3
```