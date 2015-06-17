---
layout: post
title: Never trust the row names of a dataframe in R
date: 2015-06-16
categories: [R, dataframe, row names]
---

### Introduction 
Dataframes in R have both column names and row names. Column names, which are used frequently, give the dataframes in R their characteristic distinction.
Row names, on the other hand, are rarely used. Usually, row names appear to be the same as row numbers but this is not the case. This quick blog post demonstrates that row names are not the same as row numbers. This is something most experienced R users are well aware of.

```
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


### A Tricky Situation
Let's say we are working with R in the interactive mode. We have a dataframe that we are inspecting. We want to know various things about the dataframe such as how many rows and columns it has. We can print the dataframe like this:

```
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

```
> nrow(cars)
[1] 50
```

Now, let's look at another dataframe. 

```
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

```
> nrow(cars[-18, ])
[1] 49
```

Row names are **not** row numbers. We should always use `nrow()` to determine the number of rows in a dataframe. Row names can be misleading, especially for large dataframes. In fact, row names can be modified almost like column names. 


### Behavior of Row Names
These examples demonstrate the behavior of row names.

1. Row names may be assigned any unique character or numeric values.

    ```
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

2. Attempt to set non-unique row names fails.

    ```
    > rownames(df) <- rep(1, 5)
    Error in `row.names<-.data.frame`(`*tmp*`, value = value) :
      duplicate 'row.names' are not allowed
    In addition: Warning message:
    non-unique value when setting 'row.names': ‘1’
    ```

3. Row names cannot be assigned `NA`.

    ```
    > df <- cars[1, ]
    > df
      speed dist
    1     4    2
    > rownames(df)[1] <- NA
    Error in `row.names<-.data.frame`(`*tmp*`, value = value) :
      missing values in 'row.names' are not allowed
    ```


4. Output of `rownames()` is character, even if we set numeric row names. 

    ```
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

5. Row names may be reset by assigning NULL.

    ```
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

    ```
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

    ```
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

    ```
    > df <- cars[1:5, ]
    > dump("df", "")
    df <-
    structure(list(speed = c(4, 4, 7, 7, 8), dist = c(2, 10, 4, 22,
    16)), .Names = c("speed", "dist"), row.names = c(NA, 5L), class = "data.frame")
    ```


    `dump()` _serializes_ the object and prints it as a string. We can clearly see that row names are stored in the `row.names` attribute of the object.

    In the above example, it appears that row names are all `NA`.

    **Interesting!**

    So, row names cannot be assigned `NA` (as we saw in the examples earlier), but they seem to be stored as `NA`s.

    Another function that lets us look at the internals of an R object is `attributes()`.

    ```
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

    ```
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

8. Creating a new dataframe by extraction does not automatically reset row names. Always remember to reset row names by assigning to NULL.

    ```
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

    ```
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

### Conclusion
From the above behavior of row names, we can see that row names should not really be trusted. 

1. To determine the number of rows, use `nrow()`.
2. Do not use the row names as row index to access the dataframe.
3. It's always safer to reset the row names by setting the `rownames()` to `NULL`.
4. 