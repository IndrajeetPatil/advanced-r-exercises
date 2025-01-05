# Vectors



## Atomic vectors (Exercises 3.2.5)

**Q1.** How do you create raw and complex scalars? (See `?raw` and `?complex`.)

**A1.** In R, scalars are nothing but vectors of length 1, and can be created using the same constructor.

- Raw vectors

The raw type holds raw bytes, and can be created using `charToRaw()`. For example,


``` r
x <- "A string"

(y <- charToRaw(x))
#> [1] 41 20 73 74 72 69 6e 67

typeof(y)
#> [1] "raw"
```

An alternative is to use `as.raw()`:


``` r
as.raw("–") # en-dash
#> Warning: NAs introduced by coercion
#> Warning: out-of-range values treated as 0 in coercion to
#> raw
#> [1] 00
as.raw("—") # em-dash
#> Warning: NAs introduced by coercion
#> Warning: out-of-range values treated as 0 in coercion to
#> raw
#> [1] 00
```

- Complex vectors

Complex vectors are used to represent (surprise!) complex numbers.

Example of a complex scalar:


``` r
(x <- complex(length.out = 1, real = 1, imaginary = 8))
#> [1] 1+8i

typeof(x)
#> [1] "complex"
```

**Q2.** Test your knowledge of the vector coercion rules by predicting the output of the following uses of `c()`:


``` r
c(1, FALSE)
c("a", 1)
c(TRUE, 1L)
```

**A2.** The vector coercion rules dictate that the data type with smaller size will be converted to data type with bigger size.


``` r
c(1, FALSE)
#> [1] 1 0

c("a", 1)
#> [1] "a" "1"

c(TRUE, 1L)
#> [1] 1 1
```

**Q3.** Why is `1 == "1"` true? Why is `-1 < FALSE` true? Why is `"one" < 2` false?

**A3.** The coercion rules for vectors reveal why some of these comparisons return the results that they do.


``` r
1 == "1"
#> [1] TRUE

c(1, "1")
#> [1] "1" "1"
```


``` r
-1 < FALSE
#> [1] TRUE

c(-1, FALSE)
#> [1] -1  0
```


``` r
"one" < 2
#> [1] FALSE

c("one", 2)
#> [1] "one" "2"

sort(c("one", 2))
#> [1] "2"   "one"
```

**Q4.** Why is the default missing value, `NA`, a logical vector? What's special about logical vectors? (Hint: think about `c(FALSE, NA_character_)`.)

**A4.** The `"logical"` type is the lowest in the coercion hierarchy.

So `NA` defaulting to any other type (e.g. `"numeric"`) would mean that any time there is a missing element in a vector, rest of the elements would be converted to a type higher in hierarchy, which would be problematic for types lower in hierarchy.


``` r
typeof(NA)
#> [1] "logical"

c(FALSE, NA_character_)
#> [1] "FALSE" NA
```

**Q5.** Precisely what do `is.atomic()`, `is.numeric()`, and `is.vector()` test for?

**A5.** Let's discuss them one-by-one.

- `is.atomic()`

This function checks if the object is a vector of atomic *type* (or `NULL`).

Quoting docs:

> `is.atomic` is true for the atomic types ("logical", "integer", "numeric", "complex", "character" and "raw") and `NULL`.


``` r
is.atomic(NULL)
#> [1] FALSE

is.atomic(list(NULL))
#> [1] FALSE
```

- `is.numeric()`

Its documentation says:

> `is.numeric` should only return true if the base type of the class is `double` or `integer` and values can reasonably be regarded as `numeric`

Therefore, this function only checks for `double` and `integer` base types and not other types based on top of these types (`factor`, `Date`, `POSIXt`, or `difftime`).


``` r
is.numeric(1L)
#> [1] TRUE

is.numeric(factor(1L))
#> [1] FALSE
```

- `is.vector()`

As per its documentation:

> `is.vector` returns `TRUE` if `x` is a vector of the specified mode having no attributes *other than names*. It returns `FALSE` otherwise.

Thus, the function can be incorrectif the object has attributes other than `names`.


``` r
x <- c("x" = 1, "y" = 2)

is.vector(x)
#> [1] TRUE

attr(x, "m") <- "abcdef"

is.vector(x)
#> [1] FALSE
```

A better way to check for a vector:


``` r
is.null(dim(x))
#> [1] TRUE
```

## Attributes (Exercises 3.3.4)

**Q1.** How is `setNames()` implemented? How is `unname()` implemented? Read the source code.

**A1.** Let's have a look at implementations for these functions.

- `setNames()`


``` r
setNames
#> function (object = nm, nm) 
#> {
#>     names(object) <- nm
#>     object
#> }
#> <bytecode: 0x55b735da56c0>
#> <environment: namespace:stats>
```

Given this function signature, we can see why, when no first argument is given, the result is still a named vector.


``` r
setNames(, c("a", "b"))
#>   a   b 
#> "a" "b"

setNames(c(1, 2), c("a", "b"))
#> a b 
#> 1 2
```

- `unname()`


``` r
unname
#> function (obj, force = FALSE) 
#> {
#>     if (!is.null(names(obj))) 
#>         names(obj) <- NULL
#>     if (!is.null(dimnames(obj)) && (force || !is.data.frame(obj))) 
#>         dimnames(obj) <- NULL
#>     obj
#> }
#> <bytecode: 0x55b735412b88>
#> <environment: namespace:base>
```

`unname()` removes existing names (or dimnames) by setting them to `NULL`.


``` r
unname(setNames(, c("a", "b")))
#> [1] "a" "b"
```

**Q2.** What does `dim()` return when applied to a 1-dimensional vector? When might you use `NROW()` or `NCOL()`?

**A2.** Dimensions for a 1-dimensional vector are `NULL`. For example,


``` r
dim(c(1, 2))
#> NULL
```


`NROW()` and `NCOL()` are helpful for getting dimensions for 1D vectors by treating them as if they were matrices or dataframes.


``` r
# example-1
x <- character(0)

dim(x)
#> NULL

nrow(x)
#> NULL
NROW(x)
#> [1] 0

ncol(x)
#> NULL
NCOL(x)
#> [1] 1

# example-2
y <- 1:4

dim(y)
#> NULL

nrow(y)
#> NULL
NROW(y)
#> [1] 4

ncol(y)
#> NULL
NCOL(y)
#> [1] 1
```

**Q3.** How would you describe the following three objects? What makes them different from `1:5`?


``` r
x1 <- array(1:5, c(1, 1, 5))
x2 <- array(1:5, c(1, 5, 1))
x3 <- array(1:5, c(5, 1, 1))
```

**A3.** `x1`, `x2`, and `x3` are one-dimensional **array**s, but with different "orientations", if we were to mentally visualize them. 

`x1` has 5 entries in the third dimension, `x2` in the second dimension, while `x1` in the first dimension.

**Q4.** An early draft used this code to illustrate `structure()`:


``` r
structure(1:5, comment = "my attribute")
#> [1] 1 2 3 4 5
```

But when you print that object you don't see the comment attribute. Why? Is the attribute missing, or is there something else special about it? (Hint: try using help.)

**A4.** From `?attributes` (emphasis mine):

> Note that some attributes (namely class, **comment**, dim, dimnames, names, row.names and tsp) are treated specially and have restrictions on the values which can be set.


``` r
structure(1:5, x = "my attribute")
#> [1] 1 2 3 4 5
#> attr(,"x")
#> [1] "my attribute"

structure(1:5, comment = "my attribute")
#> [1] 1 2 3 4 5
```

## S3 atomic vectors (Exercises 3.4.5)

**Q1.** What sort of object does `table()` return? What is its type? What attributes does it have? How does the dimensionality change as you tabulate more variables?

**A1.** `table()` returns an array of `integer` type and its dimensions scale with the number of variables present.


``` r
(x <- table(mtcars$am))
#> 
#>  0  1 
#> 19 13
(y <- table(mtcars$am, mtcars$cyl))
#>    
#>      4  6  8
#>   0  3  4 12
#>   1  8  3  2
(z <- table(mtcars$am, mtcars$cyl, mtcars$vs))
#> , ,  = 0
#> 
#>    
#>      4  6  8
#>   0  0  0 12
#>   1  1  3  2
#> 
#> , ,  = 1
#> 
#>    
#>      4  6  8
#>   0  3  4  0
#>   1  7  0  0

# type
purrr::map(list(x, y, z), typeof)
#> [[1]]
#> [1] "integer"
#> 
#> [[2]]
#> [1] "integer"
#> 
#> [[3]]
#> [1] "integer"

# attributes
purrr::map(list(x, y, z), attributes)
#> [[1]]
#> [[1]]$dim
#> [1] 2
#> 
#> [[1]]$dimnames
#> [[1]]$dimnames[[1]]
#> [1] "0" "1"
#> 
#> 
#> [[1]]$class
#> [1] "table"
#> 
#> 
#> [[2]]
#> [[2]]$dim
#> [1] 2 3
#> 
#> [[2]]$dimnames
#> [[2]]$dimnames[[1]]
#> [1] "0" "1"
#> 
#> [[2]]$dimnames[[2]]
#> [1] "4" "6" "8"
#> 
#> 
#> [[2]]$class
#> [1] "table"
#> 
#> 
#> [[3]]
#> [[3]]$dim
#> [1] 2 3 2
#> 
#> [[3]]$dimnames
#> [[3]]$dimnames[[1]]
#> [1] "0" "1"
#> 
#> [[3]]$dimnames[[2]]
#> [1] "4" "6" "8"
#> 
#> [[3]]$dimnames[[3]]
#> [1] "0" "1"
#> 
#> 
#> [[3]]$class
#> [1] "table"
```

**Q2.** What happens to a factor when you modify its levels? 


``` r
f1 <- factor(letters)
levels(f1) <- rev(levels(f1))
```

**A2.** Its levels change but the underlying integer values remain the same.


``` r
f1 <- factor(letters)
f1
#>  [1] a b c d e f g h i j k l m n o p q r s t u v w x y z
#> 26 Levels: a b c d e f g h i j k l m n o p q r s t u ... z
as.integer(f1)
#>  [1]  1  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17 18
#> [19] 19 20 21 22 23 24 25 26

levels(f1) <- rev(levels(f1))
f1
#>  [1] z y x w v u t s r q p o n m l k j i h g f e d c b a
#> 26 Levels: z y x w v u t s r q p o n m l k j i h g f ... a
as.integer(f1)
#>  [1]  1  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17 18
#> [19] 19 20 21 22 23 24 25 26
```

**Q3.** What does this code do? How do `f2` and `f3` differ from `f1`?


``` r
f2 <- rev(factor(letters))
f3 <- factor(letters, levels = rev(letters))
```

**A3.** In this code:

- `f2`: Only the underlying integers are reversed, but levels remain unchanged.


``` r
f2 <- rev(factor(letters))
f2
#>  [1] z y x w v u t s r q p o n m l k j i h g f e d c b a
#> 26 Levels: a b c d e f g h i j k l m n o p q r s t u ... z
as.integer(f2)
#>  [1] 26 25 24 23 22 21 20 19 18 17 16 15 14 13 12 11 10  9
#> [19]  8  7  6  5  4  3  2  1
```

- `f3`: Both the levels and the underlying integers are reversed.


``` r
f3 <- factor(letters, levels = rev(letters))
f3
#>  [1] a b c d e f g h i j k l m n o p q r s t u v w x y z
#> 26 Levels: z y x w v u t s r q p o n m l k j i h g f ... a
as.integer(f3)
#>  [1] 26 25 24 23 22 21 20 19 18 17 16 15 14 13 12 11 10  9
#> [19]  8  7  6  5  4  3  2  1
```

## Lists (Exercises 3.5.4)

**Q1.** List all the ways that a list differs from an atomic vector.

**A1.** Here is a table of comparison:

| feature                        | atomic vector                                    | list (aka generic vector)                              |
| :----------------------------- | :----------------------------------------------- | :----------------------------------------------------- |
| element type                   | unique                                           | mixed^[a list can contain a mix of types]              |
| recursive?                     | no                                               | yes^[a list can contain itself]                        |
| return for out-of-bounds index | `NA`                                             | `NULL`                                                 |
| memory address                 | single memory reference^[`lobstr::ref(c(1, 2))`] | reference per list element^[`lobstr::ref(list(1, 2))`] |

**Q2.** Why do you need to use `unlist()` to convert a list to an atomic vector? Why doesn't `as.vector()` work? 

**A2.** A list already *is* a (generic) vector, so `as.vector()` is not going to change anything, and there is no `as.atomic.vector`. Thus, we need to use `unlist()`.


``` r
x <- list(a = 1, b = 2)

is.vector(x)
#> [1] TRUE
is.atomic(x)
#> [1] FALSE

# still a list
as.vector(x)
#> $a
#> [1] 1
#> 
#> $b
#> [1] 2

# now a vector
unlist(x)
#> a b 
#> 1 2
```

**Q3.** Compare and contrast `c()` and `unlist()` when combining a date and date-time into a single vector.

**A3.** Let's first create a date and datetime object


``` r
date <- as.Date("1947-08-15")
datetime <- as.POSIXct("1950-01-26 00:01", tz = "UTC")
```

And check their attributes and underlying `double` representation:


``` r
attributes(date)
#> $class
#> [1] "Date"
attributes(datetime)
#> $class
#> [1] "POSIXct" "POSIXt" 
#> 
#> $tzone
#> [1] "UTC"

as.double(date) # number of days since the Unix epoch 1970-01-01
#> [1] -8175
as.double(datetime) # number of seconds since then
#> [1] -628991940
```

- Behavior with `c()`

Since `S3` method for `c()` dispatches on the first argument, the resulting class of the vector is going to be the same as the first argument. Because of this, some attributes will be lost.


``` r
c(date, datetime)
#> [1] "1947-08-15" "1950-01-26"

attributes(c(date, datetime))
#> $class
#> [1] "Date"

c(datetime, date)
#> [1] "1950-01-26 00:01:00 UTC" "1947-08-15 00:00:00 UTC"

attributes(c(datetime, date))
#> $class
#> [1] "POSIXct" "POSIXt" 
#> 
#> $tzone
#> [1] "UTC"
```

- Behavior with `unlist()`

It removes all attributes and we are left only with the underlying double representations of these objects.


``` r
unlist(list(date, datetime))
#> [1]      -8175 -628991940

unlist(list(datetime, date))
#> [1] -628991940      -8175
```

## Data frames and tibbles (Exercises 3.6.8)

**Q1.** Can you have a data frame with zero rows? What about zero columns?

**A1.** Data frame with 0 rows is possible. This is basically a list with a vector of length 0.


``` r
data.frame(x = numeric(0))
#> [1] x
#> <0 rows> (or 0-length row.names)
```

Data frame with 0 columns is also possible. This will be an empty list.


``` r
data.frame(row.names = 1)
#> data frame with 0 columns and 1 row
```

And, finally, data frame with 0 rows *and* columns is also possible:


``` r
data.frame()
#> data frame with 0 columns and 0 rows

dim(data.frame())
#> [1] 0 0
```

Although, it might not be common to *create* such data frames, they can be results of subsetting. For example,


``` r
BOD[0, ]
#> [1] Time   demand
#> <0 rows> (or 0-length row.names)

BOD[, 0]
#> data frame with 0 columns and 6 rows

BOD[0, 0]
#> data frame with 0 columns and 0 rows
```

**Q2.** What happens if you attempt to set rownames that are not unique?

**A2.** If you attempt to set data frame rownames that are not unique, it will not work.


``` r
data.frame(row.names = c(1, 1))
#> Error in data.frame(row.names = c(1, 1)): duplicate row.names: 1
```

**Q3.** If `df` is a data frame, what can you say about `t(df)`, and `t(t(df))`? Perform some experiments, making sure to try different column types.

**A3.** Transposing a data frame:

- transforms it into a matrix 
- coerces all its elements to be of the same type


``` r
# original
(df <- head(iris))
#>   Sepal.Length Sepal.Width Petal.Length Petal.Width Species
#> 1          5.1         3.5          1.4         0.2  setosa
#> 2          4.9         3.0          1.4         0.2  setosa
#> 3          4.7         3.2          1.3         0.2  setosa
#> 4          4.6         3.1          1.5         0.2  setosa
#> 5          5.0         3.6          1.4         0.2  setosa
#> 6          5.4         3.9          1.7         0.4  setosa

# transpose
t(df)
#>              1        2        3        4        5       
#> Sepal.Length "5.1"    "4.9"    "4.7"    "4.6"    "5.0"   
#> Sepal.Width  "3.5"    "3.0"    "3.2"    "3.1"    "3.6"   
#> Petal.Length "1.4"    "1.4"    "1.3"    "1.5"    "1.4"   
#> Petal.Width  "0.2"    "0.2"    "0.2"    "0.2"    "0.2"   
#> Species      "setosa" "setosa" "setosa" "setosa" "setosa"
#>              6       
#> Sepal.Length "5.4"   
#> Sepal.Width  "3.9"   
#> Petal.Length "1.7"   
#> Petal.Width  "0.4"   
#> Species      "setosa"

# transpose of a transpose
t(t(df))
#>   Sepal.Length Sepal.Width Petal.Length Petal.Width
#> 1 "5.1"        "3.5"       "1.4"        "0.2"      
#> 2 "4.9"        "3.0"       "1.4"        "0.2"      
#> 3 "4.7"        "3.2"       "1.3"        "0.2"      
#> 4 "4.6"        "3.1"       "1.5"        "0.2"      
#> 5 "5.0"        "3.6"       "1.4"        "0.2"      
#> 6 "5.4"        "3.9"       "1.7"        "0.4"      
#>   Species 
#> 1 "setosa"
#> 2 "setosa"
#> 3 "setosa"
#> 4 "setosa"
#> 5 "setosa"
#> 6 "setosa"

# is it a dataframe?
is.data.frame(df)
#> [1] TRUE
is.data.frame(t(df))
#> [1] FALSE
is.data.frame(t(t(df)))
#> [1] FALSE

# check type
typeof(df)
#> [1] "list"
typeof(t(df))
#> [1] "character"
typeof(t(t(df)))
#> [1] "character"

# check dimensions
dim(df)
#> [1] 6 5
dim(t(df))
#> [1] 5 6
dim(t(t(df)))
#> [1] 6 5
```

**Q4.** What does `as.matrix()` do when applied to a data frame with columns of different types? How does it differ from `data.matrix()`?

**A4.** The return type of `as.matrix()` depends on the data frame column types.

As docs for `as.matrix()` mention:

> The method for data frames will return a character matrix if there is only atomic columns and any non-(numeric/logical/complex) column, applying as.vector to factors and format to other non-character columns. Otherwise the usual coercion hierarchy (logical < integer < double < complex) will be used, e.g. all-logical data frames will be coerced to a logical matrix, mixed logical-integer will give an integer matrix, etc.

Let's experiment:


``` r
# example with mixed types (coerced to character)
(df <- head(iris))
#>   Sepal.Length Sepal.Width Petal.Length Petal.Width Species
#> 1          5.1         3.5          1.4         0.2  setosa
#> 2          4.9         3.0          1.4         0.2  setosa
#> 3          4.7         3.2          1.3         0.2  setosa
#> 4          4.6         3.1          1.5         0.2  setosa
#> 5          5.0         3.6          1.4         0.2  setosa
#> 6          5.4         3.9          1.7         0.4  setosa

as.matrix(df)
#>   Sepal.Length Sepal.Width Petal.Length Petal.Width
#> 1 "5.1"        "3.5"       "1.4"        "0.2"      
#> 2 "4.9"        "3.0"       "1.4"        "0.2"      
#> 3 "4.7"        "3.2"       "1.3"        "0.2"      
#> 4 "4.6"        "3.1"       "1.5"        "0.2"      
#> 5 "5.0"        "3.6"       "1.4"        "0.2"      
#> 6 "5.4"        "3.9"       "1.7"        "0.4"      
#>   Species 
#> 1 "setosa"
#> 2 "setosa"
#> 3 "setosa"
#> 4 "setosa"
#> 5 "setosa"
#> 6 "setosa"

str(as.matrix(df))
#>  chr [1:6, 1:5] "5.1" "4.9" "4.7" "4.6" "5.0" "5.4" ...
#>  - attr(*, "dimnames")=List of 2
#>   ..$ : chr [1:6] "1" "2" "3" "4" ...
#>   ..$ : chr [1:5] "Sepal.Length" "Sepal.Width" "Petal.Length" "Petal.Width" ...

# another example (no such coercion)
BOD
#>   Time demand
#> 1    1    8.3
#> 2    2   10.3
#> 3    3   19.0
#> 4    4   16.0
#> 5    5   15.6
#> 6    7   19.8

as.matrix(BOD)
#>      Time demand
#> [1,]    1    8.3
#> [2,]    2   10.3
#> [3,]    3   19.0
#> [4,]    4   16.0
#> [5,]    5   15.6
#> [6,]    7   19.8
```

On the other hand, `data.matrix()` always returns a numeric matrix.

From documentation of `data.matrix()`:

> Return the matrix obtained by converting all the variables in a data frame to numeric mode and then binding them together as the columns of a matrix. Factors and ordered factors are replaced by their internal codes. [...] Character columns are first converted to factors and then to integers.

Let's experiment:


``` r
data.matrix(df)
#>   Sepal.Length Sepal.Width Petal.Length Petal.Width Species
#> 1          5.1         3.5          1.4         0.2       1
#> 2          4.9         3.0          1.4         0.2       1
#> 3          4.7         3.2          1.3         0.2       1
#> 4          4.6         3.1          1.5         0.2       1
#> 5          5.0         3.6          1.4         0.2       1
#> 6          5.4         3.9          1.7         0.4       1

str(data.matrix(df))
#>  num [1:6, 1:5] 5.1 4.9 4.7 4.6 5 5.4 3.5 3 3.2 3.1 ...
#>  - attr(*, "dimnames")=List of 2
#>   ..$ : chr [1:6] "1" "2" "3" "4" ...
#>   ..$ : chr [1:5] "Sepal.Length" "Sepal.Width" "Petal.Length" "Petal.Width" ...
```

## Session information


``` r
sessioninfo::session_info(include_base = TRUE)
#> ─ Session info ───────────────────────────────────────────
#>  setting  value
#>  version  R version 4.4.2 (2024-10-31)
#>  os       Ubuntu 22.04.5 LTS
#>  system   x86_64, linux-gnu
#>  ui       X11
#>  language (EN)
#>  collate  C.UTF-8
#>  ctype    C.UTF-8
#>  tz       UTC
#>  date     2025-01-05
#>  pandoc   3.6.1 @ /opt/hostedtoolcache/pandoc/3.6.1/x64/ (via rmarkdown)
#> 
#> ─ Packages ───────────────────────────────────────────────
#>  package     * version date (UTC) lib source
#>  base        * 4.4.2   2024-10-31 [3] local
#>  bookdown      0.41    2024-10-16 [1] RSPM
#>  cli           3.6.3   2024-06-21 [1] RSPM
#>  compiler      4.4.2   2024-10-31 [3] local
#>  datasets    * 4.4.2   2024-10-31 [3] local
#>  digest        0.6.37  2024-08-19 [1] RSPM
#>  emoji         16.0.0  2024-10-28 [1] RSPM
#>  evaluate      1.0.1   2024-10-10 [1] RSPM
#>  fastmap       1.2.0   2024-05-15 [1] RSPM
#>  glue          1.8.0   2024-09-30 [1] RSPM
#>  graphics    * 4.4.2   2024-10-31 [3] local
#>  grDevices   * 4.4.2   2024-10-31 [3] local
#>  htmltools     0.5.8.1 2024-04-04 [1] RSPM
#>  knitr         1.49    2024-11-08 [1] RSPM
#>  lifecycle     1.0.4   2023-11-07 [1] RSPM
#>  magrittr    * 2.0.3   2022-03-30 [1] RSPM
#>  methods     * 4.4.2   2024-10-31 [3] local
#>  purrr         1.0.2   2023-08-10 [1] RSPM
#>  rlang         1.1.4   2024-06-04 [1] RSPM
#>  rmarkdown     2.29    2024-11-04 [1] RSPM
#>  sessioninfo   1.2.2   2021-12-06 [1] RSPM
#>  stats       * 4.4.2   2024-10-31 [3] local
#>  stringi       1.8.4   2024-05-06 [1] RSPM
#>  stringr       1.5.1   2023-11-14 [1] RSPM
#>  tools         4.4.2   2024-10-31 [3] local
#>  utils       * 4.4.2   2024-10-31 [3] local
#>  vctrs         0.6.5   2023-12-01 [1] RSPM
#>  xfun          0.49    2024-10-31 [1] RSPM
#>  yaml          2.3.10  2024-07-26 [1] RSPM
#> 
#>  [1] /home/runner/work/_temp/Library
#>  [2] /opt/R/4.4.2/lib/R/site-library
#>  [3] /opt/R/4.4.2/lib/R/library
#> 
#> ──────────────────────────────────────────────────────────
```
