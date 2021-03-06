---
title: "Indexing"
output: md_document
permalink: /indexing/
questions:
  - "How can I store values of different types in a single data structure?"
  - "How can I index things?"
  - "How can I access values by name in a data structure?"
  - "How can I create a matrix?"
objectives:
  - "Explain the difference between a list and a vector."
  - "Explain the difference between indexing with `[` and with `[[`."
  - "Use `[` and `[[` correctly to extract elements and sub-structures from data structures in R."
  - "Create a named list in R."
  - "Access elements by name using both `[` and `$` notation."
  - "Correctly identify cases in which back-quoting is necessary when accessing elements via `$`."
  - "Create and index matrices in R."
keypoints:
  - "A list is a heterogeneous vector capable of storing values of any type (including other lists)."
  - "Indexing with `[` returns a structure of the same type as the structure being indexed (e.g., returns a list when applied to a list)."
  - "Indexing with `[[` strips away one level of structure (i.e., returns the indicated element without any wrapping)."
  - "Use `list('name' = value, ...)` to name the elements of a list."
  - "Use either `L['name']` or `L$name` to access elements by name."
  - "Use back-quotes around the name with `$` notation if the name is not a legal R variable name."
  - "Use `matrix(values, nrow = N)` to create a matrix with `N` rows containing the given values."
  - "Use `m[i, j]` to get the value at the i'th row and j'th column of a matrix."
  - "Use `m[i,]` to get a vector containing the values in the i'th row of a matrix."
  - "Use `m[,j]` to get a vector containing the values in the j'th column of a matrix."
---



One of the things that newcomers to R often trip over is the various ways in which structures can be indexed.
All of the following are legal:


```r
thing[i]
thing[i, j]
thing[[i]]
thing[[i, j]]
thing$name
thing$"name"
```

but they can behave differently depending on what kind of thing `thing` is.
To explain, we must first take a look at lists.

## How can I store a mix of different types of objects?

A list in R is a vector that can contain values of many different types.
(The technical term for this is [heterogeneous](../glossary/#heterogeneous),
in contrast with the [homogeneous](../glossary/#homogeneous) nature of vectors' values.)
We'll use this list in our examples:


```r
thing <- list("first", c(2, 20, 200), 30)
thing
#> [[1]]
#> [1] "first"
#> 
#> [[2]]
#> [1]   2  20 200
#> 
#> [[3]]
#> [1] 30
```

The output tells us that the first element of `thing` is a vector of one element,
that the second is a vector of three elements,
and the third is again a vector of one element.

## What is the difference between `[` and `[[`?

The output above strongly suggests that we can get the elements of a list using `[[` (double square brackets):


```r
thing[[1]]
#> [1] "first"
thing[[2]]
#> [1]   2  20 200
thing[[3]]
#> [1] 30
```

Let's have a look at the types of those three values:


```r
typeof(thing[[1]])
#> [1] "character"
typeof(thing[[2]])
#> [1] "double"
typeof(thing[[3]])
#> [1] "double"
```

Good: they are vectors.
What do we get if we use single square brackets `[`?


```r
typeof(thing[1])
#> [1] "list"
```

Sure enough, the value itself is a list:


```r
thing[1]
#> [[1]]
#> [1] "first"
```

This shows the difference between `[[` and `[`:
the former peels away a layer of data structure, returning only the sub-structure,
while the latter gives us back a structure of the same type as the thing being indexed.
Since a "scalar" is just a vector of length 1,
there is no difference between the two when they are applied to vectors:


```r
v <- c("first", "second", "third")
v[2]
#> [1] "second"
typeof(v[2])
#> [1] "character"
v[[2]]
#> [1] "second"
typeof(v[[2]])
#> [1] "character"
```

> **Flattening**
>
> If a list is just a vector of objects, why do we need the function `list`?
> Why can't we create a list with `c("first", c(2, 20, 200), 30)`?
> The answer is that R flattens the arguments to `c`,
> so that `c(c(1, 2), c(3, 4))` produces `c(1, 2, 3, 4)`.
> It also does automatic type conversion:
> `c("first", c(2, 20, 200), 30)` produces a vector of character strings
> `c("first", "2", "20", "200", "30")`.

> **Recursive Indexing**
>
> Using `[[` with a list subsets recursively:
> if `thing <- list(a = list(b = list(c = list(d = 1))))`,
> then `thing[[c("a", "b", "c", "d")]]` selects the 1.

## How can I access elements by name?

R allows us to name the elements in vectors:
if we assign `c(one = 1, two = 2, three = 3)` to `names`,
then `names["two"]` is 2.
We can use this to create a lookup table:


```r
values <- c("m", "f", "u", "f", "f", "m", "m")
lookup <- c(m = "Male", f = "Female", u = "Unstated")
lookup[values]
#>          m          f          u          f          f          m 
#>     "Male"   "Female" "Unstated"   "Female"   "Female"     "Male" 
#>          m 
#>     "Male"
```

If the structure in question is a list rather than an atomic vector of numbers, characters, or logicals,
we can use the syntax `lookup$m` instead of `lookup["m"]`:


```r
lookup_list <- list(m = "Male", f = "Female", u = "Unstated")
lookup_list$m
#> [1] "Male"
```

We will explore this in more detail when we look at [the tidyverse](../tidyverse/),
since that is where access-by-name is used most often.
For now,
simply note that if the name of an element isn't a legal variable name,
we have to put it in backward quotes to use it as an accessor:


```r
another_list <- list("first field" = "F", "second field" = "S")
another_list$`first field`
#> [1] "F"
```

Wherever possible, it's better to choose names that don't require back-quoting,
such as `first_first`.

## How can I create and index a matrix?

Matrices are frequently used in statistics, so R provides built-in support for them.
After `a <- matrix(1:9, nrow = 3)`,
`a` is a 3x3 matrix containing the values 1 through 9.
What may surprise you is the order in which the values generated by the expression `1:9` are laid out:


```r
a <- matrix(1:9, nrow = 3)
a
#>      [,1] [,2] [,3]
#> [1,]    1    4    7
#> [2,]    2    5    8
#> [3,]    3    6    9
```

Under the hood,
a matrix is a vector with an [attribute](../glossary/#attribute) called `dim` that stores its dimensions:


```r
dim(a)
#> [1] 3 3
```

`a[3, 3]` is a vector of length 1 containing the value 9 (because scalars in R are actually vectors),
while `a[1,]` is the vector `c(1, 4, 7)` (because we are selecting the first row of the matrix)
and `a[,1]` is the vector `c(1, 2, 3)` (because we are selecting the first column of the matrix).
Elements can still be accessed using a single index,
which returns the value from that location in the underlying vector:


```r
a[8]
#> [1] 8
```

{% include links.md %}
