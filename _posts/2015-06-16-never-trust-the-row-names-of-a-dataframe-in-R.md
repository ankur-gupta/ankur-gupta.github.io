---
layout: post
title: "Never trust rownames of a dataframe"
date: 2015-06-16
update_date: 2015-08-15
categories: [R, dataframe, row names]
comments: true
---

## Introduction
Dataframes in R have both column names and row names. Column names, which are used frequently, give the dataframes in R their characteristic distinction.
Row names, on the other hand, are rarely used. Usually, row names appear to be the same as row numbers but this is not the case. This quick blog post demonstrates that row names are not the same as row numbers. This is something most experienced R users are well aware of.

```r
> df <- cars[1:5, ]
> df
  speed dist
1     4    2
2     4   10
3     7    4
4     7   22
5     8   16
> rownames(df)
[1] "1" "2" "3" "4" "5"
> colnames(df)
[1] "speed" "dist"
```

## A Tricky Situation
Let's say we are working with R in the interactive mode. We have a dataframe that we are inspecting. We want to know various things about the dataframe such as how many rows and columns it has. We can print the dataframe like this:

```r
> cars
   speed dist
1      4    2
2      4   10
...
48    24   93
49    24  120
50    25   85
>
```

We can see that `cars` has 2 columns named `speed` and `dist`. When we print the dataframe, we can see that the row names are printed as well. Looking at the rownames, it would appear that `cars` has 50 rows. Right?

This happens to be correct, in this case, as we can check using `nrow()`. But this is not true in general.

```r
> nrow(cars)
[1] 50
```

Now, let's look at another dataframe.

```r
> cars[-18, ]
   speed dist
1      4    2
2      4   10
3      7    4
...
48    24   93
49    24  120
50    25   85
```

Now, if `cars` has 50 rows, `cars[-18, ]` contains 49 rows. But, by simply looking at the printed row names, we would see that the last row still has the row name "50" but `cars[-18, ]` has only 49 rows.

```r
> nrow(cars[-18, ])
[1] 49
```

Row names are **not** row numbers. We should always use `nrow()` to determine the number of rows in a dataframe. Row names can be misleading, especially for large dataframes.

The numbers printed along side the columns of the dataframe are row names and not row numbers. What are row numbers and how do we distinguish row numbers from row names? The following helpful tip from [Ryan Garner](https://www.linkedin.com/in/ryanstevengarner) can help us answer this question.

### Row names vs Row numbers
Let's rearrange the rows of `cars` using `order()` function.

```r
> df <- cars[order(cars$dist), ]
> df
   speed dist
1      4    2
3      7    4
2      4   10
6      9   10
12    12   14
...
47    24   92
48    24   93
49    24  120
```

The numbers `1, 3, 2, 6, 12 ..., 47, 47, 49` are row names. Row numbers (or row indices) are simply the consecutive sequence of integers from `1` to `nrow(df)`. Row numbers are **never** printed along side the dataframe. The second row (i.e. row number `2`) of `df` in this example may be obtained by

```r
> df[2, ] # Row number 2
  speed dist
3     7    4
```

As we can see, the second row (i.e. row number `2`) has row name `"3"`. We can obtain the same row of `df` by using the appropriate row name

```r
> df["3", ]  # Row name "3"
  speed dist
3     7    4
```

However, the row name `"2"` is **not** the same as row number `2`.

```r
> df["2", ]  # Row name "2"
  speed dist
2     4   10
```

In the construct, `df[x, ]`, we access either the row number or the row name depending upon the class of `x`. If `x` is `numeric` or `integer`, we access the row number `x`. If `x` is `character`, we access the row name `x`. It is easy to confuse row numbers for row names and vice versa.

### Row logicals vs Row numbers vs Row names
(Update August 15, 2015)

There is yet another way to index the rows of a dataframe -- using logicals.

```r
> df <- cars[1:5, ]
> df[rep(TRUE, 5), ]
  speed dist
1     4    2
2     4   10
3     7    4
4     7   22
5     8   16
> df[rep(FALSE, 5), ]
[1] speed dist
<0 rows> (or 0-length row.names)
```

Each row of the dataframe is selected if there is a corresponding `TRUE` value for that row.
If the corresponding value is `FALSE`, then that row is not selected. This is logical indexing.
It seems easy enough and very useful too, as shown below:

```r
> df$speed == 4
[1]  TRUE  TRUE FALSE FALSE FALSE
> df[df$speed == 4, ]
  speed dist
1     4    2
2     4   10
```
Things become complicated when we start combining logical indexing with R's vector recycling
behavior. What would happen if the logical vector we use to index the rows of `df` is
shorter than `5` ?

```r
> df[TRUE, ]
  speed dist
1     4    2
2     4   10
3     7    4
4     7   22
5     8   16
```

On the face of it, `df[TRUE, ]` makes no sense. `df` has 5 rows and we have only one logical
index (_i.e._ `TRUE`). Which row of `df` should be selected?

_R could throw an error!_ But, R is forgiving. It doesn't throw an error. Instead,
R resolves this problem by employing vector recycling rules. The logical vector `TRUE` is
repeated 5 times such that each row has
`df[TRUE, ]` produces the same result as `df[rep(TRUE, 5), ]`. Let's look at another example.

```r
> df[c(TRUE, FALSE), ]
  speed dist
1     4    2
3     7    4
5     8   16
```

Now, the vector `c(TRUE, FALSE)` is repeated, which creates the effect of basically selecting
every odd row of the dataframe `df`. For some users, this might even be useful.

So, along with the row number indexing, row name indexing, we also have logical indexing.
The kind of indexing used depends on the type of index. Row name indexing is used if the
index is `numeric` (or `integer`), row name indexing is used when the index is `character` and
logical indexing is used when the index is logical. Another distinction is that only logical indexing employs vector recycling (if needed). Row name indexing and row number indexing do not use recycling (as shown in the next section).

### What about missing values?
R has missing values in every data type.

```r
> class(NA)
[1] "logical"
> class(as.character(NA))
[1] "character"
> class(as.numeric(NA))
[1] "numeric"
> class(as.factor(NA))
[1] "factor"
```

By default, `NA` is a logical. So, what happens when we row-index a dataframe using `NA` ?
It depends on the type of `NA`.

```r
> df[NA, ]
     speed dist
NA      NA   NA
NA.1    NA   NA
NA.2    NA   NA
NA.3    NA   NA
NA.4    NA   NA
```

Since `NA` is logical by default, logical indexing (with vector recycling) is used in the above example.

However, if we use row number indexing or row name indexing:

```r
> df[as.numeric(NA), ]
   speed dist
NA    NA   NA
> df[as.character(NA), ]
   speed dist
NA    NA   NA
```

Again, using `as.numeric(NA)` to index a dataframe makes no sense. Are we trying to obtain a missing row of the dataframe? R resolves this by returning rows full of `NA`s. No vector recycling is done, which means that we get back one row back for each element in the index.

```r
> df[rep(as.character(NA), 3), ]
     speed dist
NA      NA   NA
NA.1    NA   NA
NA.2    NA   NA
```
(Thanks to [Ryan Brady](https://www.linkedin.com/in/rdbrady) for pointing out this issue).


This example naturally leads us to a question about uniqueness of row names.
Are row names necessarily unique? The answer is **yes**, as we will see in the next section. Row names, much like column names, can be modified as we can see in the next section. However, row names differ from column names in one aspect - row names are always unique (in contrast, two columns in a dataframe may have the exact same name). Uniqueness of row names ensures that `df["2", ]` will return a dataframe with exactly one row (even if the row name `"2"` does not exist but more on that in another post).

## Behavior of Row Names
These examples demonstrate the behavior of row names.

1. Row names may be assigned any unique character or numeric values.

    ```r
    > df <- cars[1:5, ]
    > rownames(df)
    [1] "1" "2" "3" "4" "5"
    > rownames(df) <- letters[1:5]
    > df
    speed dist
    a     4    2
    b     4   10
    c     7    4
    d     7   22
    e     8   16
    > rownames(df)
    [1] "a" "b" "c" "d" "e"
    ```

2. Attempt to set non-unique row names fails. But this doesn't mean that the row names of a dataframe cannot be made non-unique (see the next section).

    ```r
    > rownames(df) <- rep(1, 5)
    Error in `row.names<-.data.frame`(`*tmp*`, value = value) :
      duplicate 'row.names' are not allowed
    In addition: Warning message:
    non-unique value when setting 'row.names': ‘1’
    ```

3. Row names cannot be **assigned** `NA`. More on `NA`s below.

    ```r
    > df <- cars[1, ]
    > df
      speed dist
    1     4    2
    > rownames(df)[1] <- NA
    Error in `row.names<-.data.frame`(`*tmp*`, value = value) :
      missing values in 'row.names' are not allowed
    ```


4. Output of `rownames()` is character, even if we set numeric row names.

    ```r
    > rownames(df) <- runif(5)
    > df
                       speed dist
    0.444560116855428      4    2
    0.0545551690738648     4   10
    0.633737650699914      7    4
    0.691298563731834      7   22
    0.784601176623255      8   16
    > rownames(df)
    [1] "0.444560116855428"  "0.0545551690738648" "0.633737650699914"
    [4] "0.691298563731834"  "0.784601176623255"
    > class(rownames(df))
    [1] "character"
    ```

5. Row names may be reset by assigning NULL. After this step, the row names are simply `character`-version of row numbers.

    ```r
    > df <- cars[-18, ]
    > rownames(df) <- NULL
    > df
       speed dist
    1      4    2
    2      4   10
    3      7    4
    ...
    47    24   93
    48    24  120
    49    25   85
    > rownames(df)
     [1] "1"  "2"  "3"  "4"  "5"  "6"  "7"  "8"  "9"  "10" "11" "12" "13" "14" "15"
    [16] "16" "17" "18" "19" "20" "21" "22" "23" "24" "25" "26" "27" "28" "29" "30"
    [31] "31" "32" "33" "34" "35" "36" "37" "38" "39" "40" "41" "42" "43" "44" "45"
    [46] "46" "47" "48" "49"
    ```

6. Row names may be set individually.

    ```r
    > df <- cars[1:5, ]
    > df
      speed dist
    1     4    2
    2     4   10
    3     7    4
    4     7   22
    5     8   16
    > rownames(df)[1] <- ""
    > df
      speed dist
          4    2
    2     4   10
    3     7    4
    4     7   22
    5     8   16
    > rownames(df)[2:4] <- letters[1:3]
    > df
      speed dist
          4    2
    a     4   10
    b     7    4
    c     7   22
    5     8   16
    ```

7. Even though row names appear to be "reset" or belong to `character` class, row names are stored _as-assigned_. Let's see this using an example.

    ```r
    > df <- cars[1:5, ]
    > str(df)
    'data.frame':   5 obs. of  2 variables:
     $ speed: num  4 4 7 7 8
     $ dist : num  2 10 4 22 16
    > rownames(df) <- letters[1:5]
    > df
      speed dist
    a     4    2
    b     4   10
    c     7    4
    d     7   22
    e     8   16
    > str(df)
    'data.frame':   5 obs. of  2 variables:
     $ speed: num  4 4 7 7 8
     $ dist : num  2 10 4 22 16
    ```

    First thing to notice is that `str()` does not show row names, even if the row names are not the default sequence of `1` to `nrow(df)`. If `str()` does not show us the row names, then how are the row names stored?

    There are two functions that can can help us answer this question: `dump()` and `attributes()`.

    ```r
    > df <- cars[1:5, ]
    > dump("df", "")
    df <-
    structure(list(speed = c(4, 4, 7, 7, 8), dist = c(2, 10, 4, 22,
    16)), .Names = c("speed", "dist"), row.names = c(NA, 5L), class = "data.frame")
    ```


    `dump()` _serializes_ the object and prints it as a string. We can clearly see that row names are stored in the `row.names` attribute of the object.

    In the above example, it appears that row names are `c(NA, 5L)`!

    **Interesting.**

    So, row names cannot be assigned `NA` (as we saw in the examples earlier), but they seem to be stored as something that contains `NA`.

    Another function that lets us look at the internals of an R object is `attributes()`.

    ```r
    > df <- cars[1:5, ]
    > attributes(df)
    $names
    [1] "speed" "dist"

    $row.names
    [1] 1 2 3 4 5

    $class
    [1] "data.frame"

    > attributes(df)$row.names
    [1] 1 2 3 4 5
    > class(attributes(df)$row.names)
    [1] "integer"
    ```

    Even more interestingly, the row names appear to be `integer` and not `NA`!

    _Oh, the quirks of R_.

    Anyways, now, let's look at a dataframe with explicitly assigned row names.

    ```r
    > df <- cars[1:5, ]
    > df
      speed dist
    1     4    2
    2     4   10
    3     7    4
    4     7   22
    5     8   16
    > rownames(df) <- letters[1:5]
    > df
      speed dist
    a     4    2
    b     4   10
    c     7    4
    d     7   22
    e     8   16
    > dump("df", "")
    df <-
    structure(list(speed = c(4, 4, 7, 7, 8), dist = c(2, 10, 4, 22,
    16)), .Names = c("speed", "dist"), row.names = c("a", "b", "c",
    "d", "e"), class = "data.frame")
    > attributes(df)
    $names
    [1] "speed" "dist"

    $row.names
    [1] "a" "b" "c" "d" "e"

    $class
    [1] "data.frame"

    > attributes(df)$row.names
    [1] "a" "b" "c" "d" "e"
    > class(attributes(df)$row.names)
    [1] "character"
    ```

    In this case, the row names remain consistent no matter how we look at them.

8. Creating a new dataframe by extraction does not automatically reset row names. If you want the row names to match the row numbers, reset row names by assigning to `NULL`.

    ```r
    > df <- cars[c(1, 18, 50), ]
    > df
       speed dist
    1      4    2
    18    13   34
    50    25   85
    > rownames(df)
    [1] "1"  "18" "50"
    ```

9. We can add two dataframes that have different row names without an error or a warning. The row names of the first operand of `+` are preserved.

    ```r
    > df <- cars[c(1, 18, 50), ]
    > df2 <- cars[c(2, 19, 35), ]
    > df
       speed dist
    1      4    2
    18    13   34
    50    25   85
    > df2
       speed dist
    2      4   10
    19    13   46
    35    18   84
    > df3 <- df + df2
    > df3
       speed dist
    1      8   12
    18    26   80
    50    43  169
    > df4 <- df2 + df
    > df4
       speed dist
    2      8   12
    19    26   80
    35    43  169
    ```

## Extremely quirky row name behavior
We saw that `rownames<-` does not allow us to specify non-unique row names.
Turns out, we can can specify non-unique row names, just not using the
`get("rownames<-")` function.

Let's look at a rather common way to create dataframes - using matrices.
This is a matrix with row names (yes, matrices can have row names too):

```r
mat <-
structure(c(1L, 2L, 4L, 5L, 7L, 8L), .Dim = 2:3, .Dimnames = list(
    c("row1", "row2"), NULL))
```

As we can see the row names of `mat` are different:

```r
> mat
     [,1] [,2] [,3]
row1    1    4    7
row2    2    5    8
```

We can `rbind` two matrices having the same row names:

```r
> rbind(mat, mat)
     [,1] [,2] [,3]
row1    1    4    7
row2    2    5    8
row1    1    4    7
row2    2    5    8
```

Now, we'd like to convert the above matrix into a dataframe:

```r
> df <- as.data.frame(rbind(mat, mat))
> dim(df)
[1] 4 3
```

Looks like we succeeded, right? `df` seems to have the correct dimensions.
Let's try printing it:

```r
> print(df)
Error in data.frame(V1 = c("1", "2", "1", "2"), V2 = c("4", "5", "4",  :
  duplicate row.names: row1, row2
```

**Error!** Turns out, that merely printing the dataframe throws an error.

Why?

Because the row names of `df` are non-unique:

```r
> dump("df", "")
df <-
structure(list(V1 = c(1L, 2L, 1L, 2L), V2 = c(4L, 5L, 4L, 5L),
    V3 = c(7L, 8L, 7L, 8L)), .Names = c("V1", "V2", "V3"), row.names = c("row1", "row2", "row1", "row2"), class = "data.frame")
```
So, we can indeed create a dataframe with non-unique rows. Such a dataframe even works in some cases (we were able to print out its dimensions). But in other cases (wherever row names are required), we encounter an error.

This example demonstrates yet another quirk of dataframe row names. This error can be really surprising and may even go undetected. Ideally, the function `as.data.frame()` should always check for duplicated row names but it doesn't. How can we correct this problem ?

One simple solution is to just reset the row names:

```r
> rownames(df) <- NULL
> print(df)
  V1 V2 V3
1  1  4  7
2  2  5  8
3  1  4  7
4  2  5  8
```

Alternatively, we can specify row names while converting a matrix to a dataframe:

```r
> df <- as.data.frame(rbind(mat, mat), row.names = 1:4)
> df
  V1 V2 V3
1  1  4  7
2  2  5  8
3  1  4  7
4  2  5  8
```

Note that many R users run into this issue in another way:

```r
> as.data.frame(rbind(mat, mat))
Error in data.frame(V1 = c("1", "2", "1", "2"), V2 = c("4", "5", "4",  :
  duplicate row.names: row1, row2
```

Instead of assigning `as.data.frame(rbind(mat, mat))` to `df` (which prevents printing), if we simply evaluate the expression on the interpreter, we see the error immediately. This error is not because the evaluation failed. This error is because the printing of the evaluated expression failed.

*Note:* It appears that matrices can have non-unique row names, but then, who knows ? May be we just haven't found the case in which R throws an error.

## Conclusions
From the above behavior of row names, we can see that row names should not really be trusted. Unfortunately, there are some R packages on CRAN that explicitly use the row names which often creates problems.

For example, let us say we need to write a function that accepts a dataframe as input and returns some purposefully selected rows as output.

```r
sample.rows <- function(df) {
    # Select some rows from df
    return(df[selected.rows, ])
}
```

In order to write the function `sample.rows()`, we might use row names in such a way that we change the row names of `df`. The output of this function may then have different row names than the input `df`. The end-user of this function might actually expect that the row names would remain the unchanged. Thus, modifying row names is a usually not a good idea and so is writing code that depends too much on row names.

Here are a few tips that might help us avoid trouble:

1. To determine the number of rows, use `nrow()`.
2. Row name is the **not** same as row number (row number = row index).
3. Avoid the use of row names. Use row numbers instead.
4. Mind the vector recycling when using logical indexing.
5. If row names must be used, always use `rownames()` instead of `attribute()`. At the very least, be consistent in the function used to extract row names.
6. Take special care when serializing dataframes using `dump()` or `dput()`.
7. Take care while converting matrices to dataframes. For robustness, reset row names.
8. When writing functions, remember that the end-user (which might be you), may expect the row names to remain unchanged.
9. Do not expect the row names to be in order. If you must use row names and you require row names to be in order, reset the row names by assigning `rownames()` to `NULL`.
