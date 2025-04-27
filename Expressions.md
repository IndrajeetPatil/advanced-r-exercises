# Expressions



Attaching the needed libraries:


``` r
library(rlang, warn.conflicts = FALSE)
library(lobstr, warn.conflicts = FALSE)
```

## Abstract syntax trees (Exercises 18.2.4)

**Q1.** Reconstruct the code represented by the trees below:


```
#> █─f 
#> └─█─g 
#>   └─█─h
#> █─`+` 
#> ├─█─`+` 
#> │ ├─1 
#> │ └─2 
#> └─3
#> █─`*` 
#> ├─█─`(` 
#> │ └─█─`+` 
#> │   ├─x 
#> │   └─y 
#> └─z
```

**A1.** Below is the reconstructed code.


``` r
f(g(h()))
1 + 2 + 3
(x + y) * z
```

We can confirm it by drawing ASTs for them:


``` r
ast(f(g(h())))
#> █─f 
#> └─█─g 
#>   └─█─h

ast(1 + 2 + 3)
#> █─`+` 
#> ├─█─`+` 
#> │ ├─1 
#> │ └─2 
#> └─3

ast((x + y) * z)
#> █─`*` 
#> ├─█─`(` 
#> │ └─█─`+` 
#> │   ├─x 
#> │   └─y 
#> └─z
```

**Q2.** Draw the following trees by hand and then check your answers with `ast()`.


``` r
f(g(h(i(1, 2, 3))))
f(1, g(2, h(3, i())))
f(g(1, 2), h(3, i(4, 5)))
```

**A2.** Successfully drawn by hand. Checking using `ast()`:


``` r
ast(f(g(h(i(1, 2, 3)))))
#> █─f 
#> └─█─g 
#>   └─█─h 
#>     └─█─i 
#>       ├─1 
#>       ├─2 
#>       └─3

ast(f(1, g(2, h(3, i()))))
#> █─f 
#> ├─1 
#> └─█─g 
#>   ├─2 
#>   └─█─h 
#>     ├─3 
#>     └─█─i

ast(f(g(1, 2), h(3, i(4, 5))))
#> █─f 
#> ├─█─g 
#> │ ├─1 
#> │ └─2 
#> └─█─h 
#>   ├─3 
#>   └─█─i 
#>     ├─4 
#>     └─5
```

**Q3.** What's happening with the ASTs below? (Hint: carefully read `?"^"`.)


``` r
ast(`x` + `y`)
#> █─`+` 
#> ├─x 
#> └─y
ast(x**y)
#> █─`^` 
#> ├─x 
#> └─y
ast(1 -> x)
#> █─`<-` 
#> ├─x 
#> └─1
```

**A3.** The `str2expression()` helps make sense of these ASTs.

The non-syntactic names are parsed to names. Thus, backticks have been removed in the AST.


``` r
str2expression("`x` + `y`")
#> expression(x + y)
```

As mentioned in the docs for `^`:

> \*\* is translated in the parser to \^


``` r
str2expression("x**y")
#> expression(x^y)
```

The rightward assignment is parsed to leftward assignment:


``` r
str2expression("1 -> x")
#> expression(x <- 1)
```

**Q4.** What is special about the AST below?


``` r
ast(function(x = 1, y = 2) {})
#> █─`function` 
#> ├─█─x = 1 
#> │ └─y = 2 
#> ├─█─`{` 
#> └─NULL
```

**A4.** As mentioned in [this](https://adv-r.hadley.nz/functions.html#fun-components) section:

> Like all objects in R, functions can also possess any number of additional `attributes()`. One attribute used by base R is `srcref`, short for source reference. It points to the source code used to create the function. The `srcref` is used for printing because, unlike `body()`, it contains code comments and other formatting.

Therefore, the last leaf in this AST, although not specified in the function call, represents source reference attribute.

**Q5.** What does the call tree of an `if` statement with multiple `else if` conditions look like? Why?

**A5.** There is nothing special about this tree. It just shows the nested loop structure inherent to code with `if` and multiple `else if` statements.


``` r
ast(if (FALSE) 1 else if (FALSE) 2 else if (FALSE) 3 else 4)
#> █─`if` 
#> ├─FALSE 
#> ├─1 
#> └─█─`if` 
#>   ├─FALSE 
#>   ├─2 
#>   └─█─`if` 
#>     ├─FALSE 
#>     ├─3 
#>     └─4
```

## Expressions (Exercises 18.3.5)

**Q1.** Which two of the six types of atomic vector can't appear in an expression? Why? Similarly, why can't you create an expression that contains an atomic vector of length greater than one?

**A1.** Out of the six types of atomic vectors, the two that can't appear in an expression are: complex and raw.

Complex numbers are created via a **function call** (using `+`), as can be seen by its AST:


``` r
x_complex <- expr(1 + 1i)
typeof(x_complex)
#> [1] "language"

ast(1 + 1i)
#> █─`+` 
#> ├─1 
#> └─1i
```

Similarly, for raw vectors (using `raw()`):


``` r
x_raw <- expr(raw(2))
typeof(x_raw)
#> [1] "language"

ast(raw(2))
#> █─raw 
#> └─2
```

Contrast this with other atomic vectors:


``` r
x_int <- expr(2L)
typeof(x_int)
#> [1] "integer"

ast(2L)
#> 2L
```

For the same reason, you can't you create an expression that contains an atomic vector of length greater than one since that itself is a function call that uses `c()` function:


``` r
x_vec <- expr(c(1, 2))
typeof(x_vec)
#> [1] "language"

ast(c(1, 2))
#> █─c 
#> ├─1 
#> └─2
```

**Q2.** What happens when you subset a call object to remove the first element? e.g. `expr(read.csv("foo.csv", header = TRUE))[-1]`. Why?

**A2.** A captured function call like the following creates a call object:


``` r
expr(read.csv("foo.csv", header = TRUE))
#> read.csv("foo.csv", header = TRUE)

typeof(expr(read.csv("foo.csv", header = TRUE)))
#> [1] "language"
```

As mentioned in the [respective section](https://adv-r.hadley.nz/expressions.html#function-position):

> The first element of the call object is the function position.

Therefore, when the first element in the call object is removed, the next one moves in the function position, and we get the observed output:


``` r
expr(read.csv("foo.csv", header = TRUE))[-1]
#> "foo.csv"(header = TRUE)
```

**Q3.** Describe the differences between the following call objects.


``` r
x <- 1:10
call2(median, x, na.rm = TRUE)
call2(expr(median), x, na.rm = TRUE)
call2(median, expr(x), na.rm = TRUE)
call2(expr(median), expr(x), na.rm = TRUE)
```

**A4.** The differences in the constructed call objects are due to the different *type* of arguments supplied to first two parameters in the `call2()` function.

Types of arguments supplied to `.fn`:


``` r
typeof(median)
#> [1] "closure"
typeof(expr(median))
#> [1] "symbol"
```

Types of arguments supplied to the dynamic dots:


``` r
x <- 1:10
typeof(x)
#> [1] "integer"
typeof(expr(x))
#> [1] "symbol"
```

The following outputs can be understood using the following properties:

- when `.fn` argument is a `closure`, that function is inlined in the constructed function call
- when `x` is not a symbol, its value is passed to the function call


``` r
x <- 1:10

call2(median, x, na.rm = TRUE)
#> (function (x, na.rm = FALSE, ...) 
#> UseMethod("median"))(1:10, na.rm = TRUE)

call2(expr(median), x, na.rm = TRUE)
#> median(1:10, na.rm = TRUE)

call2(median, expr(x), na.rm = TRUE)
#> (function (x, na.rm = FALSE, ...) 
#> UseMethod("median"))(x, na.rm = TRUE)

call2(expr(median), expr(x), na.rm = TRUE)
#> median(x, na.rm = TRUE)
```

Importantly, all of the constructed call objects evaluate to give the same result:


``` r
x <- 1:10

eval(call2(median, x, na.rm = TRUE))
#> [1] 5.5

eval(call2(expr(median), x, na.rm = TRUE))
#> [1] 5.5

eval(call2(median, expr(x), na.rm = TRUE))
#> [1] 5.5

eval(call2(expr(median), expr(x), na.rm = TRUE))
#> [1] 5.5
```

**Q4.** `call_standardise()` doesn't work so well for the following calls. Why? What makes `mean()` special?


``` r
call_standardise(quote(mean(1:10, na.rm = TRUE)))
#> Warning: `call_standardise()` is deprecated as of rlang 0.4.11
#> This warning is displayed once every 8 hours.
#> mean(x = 1:10, na.rm = TRUE)
call_standardise(quote(mean(n = T, 1:10)))
#> mean(x = 1:10, n = T)
call_standardise(quote(mean(x = 1:10, , TRUE)))
#> mean(x = 1:10, , TRUE)
```

**A4.** This is because of the ellipsis in `mean()` function signature:


``` r
mean
#> function (x, ...) 
#> UseMethod("mean")
#> <bytecode: 0x56429c8ca818>
#> <environment: namespace:base>
```

As mentioned in the respective [section](If the function uses ... it’s not possible to standardise all arguments.):

> If the function uses `...` it’s not possible to standardise all arguments.

`mean()` is an S3 generic and the dots are passed to underlying S3 methods.

So, the output can be improved using a specific method. For example:


``` r
call_standardise(quote(mean.default(n = T, 1:10)))
#> mean.default(x = 1:10, na.rm = T)
```

**Q5.** Why does this code not make sense?


``` r
x <- expr(foo(x = 1))
names(x) <- c("x", "y")
```

**A5.** This doesn't make sense because the first position in a call object is reserved for function (function position), and so assigning names to this element will just be ignored by R:


``` r
x <- expr(foo(x = 1))
x
#> foo(x = 1)

names(x) <- c("x", "y")
x
#> foo(y = 1)
```

**Q6.** Construct the expression `if(x > 1) "a" else "b"` using multiple calls to `call2()`. How does the code structure reflect the structure of the AST?

**A6.** Using multiple calls to construct the required expression:


``` r
x <- 5
call_obj1 <- call2(">", expr(x), 1)
call_obj1
#> x > 1

call_obj2 <- call2("if", cond = call_obj1, cons.expr = "a", alt.expr = "b")
call_obj2
#> if (x > 1) "a" else "b"
```

This construction follows from the prefix form of this expression, revealed by its AST:


``` r
ast(if (x > 1) "a" else "b")
#> █─`if` 
#> ├─█─`>` 
#> │ ├─x 
#> │ └─1 
#> ├─"a" 
#> └─"b"
```

## Parsing and grammar (Exercises 18.4.4)

**Q1.** R uses parentheses in two slightly different ways as illustrated by these two calls:


``` r
f((1))
`(`(1 + 1)
```

Compare and contrast the two uses by referencing the AST.

**A1.** Let's first have a look at the AST:


``` r
ast(f((1)))
#> █─f 
#> └─█─`(` 
#>   └─1
ast(`(`(1 + 1))
#> █─`(` 
#> └─█─`+` 
#>   ├─1 
#>   └─1
```

As, you can see `(` is being used in two separate ways:

- As a function in its own right ``"`(`"``
- As part of the prefix syntax (`f()`)

This is why, in the AST for `f((1))`, we see only one ``"`(`"`` (the first use case), and not for `f()`, which is part of the function syntax (the second use case).

**Q2.** `=` can also be used in two ways. Construct a simple example that shows both uses.

**A2.** Here is a simple example illustrating how `=` can also be used in two ways:

- for assignment
- for named arguments in function calls


``` r
m <- mean(x = 1)
```

We can also have a look at its AST:


``` r
ast({
  m <- mean(x = 1)
})
#> █─`{` 
#> └─█─`<-` 
#>   ├─m 
#>   └─█─mean 
#>     └─x = 1
```

**Q3.** Does `-2^2` yield 4 or -4? Why?

**A3.** The expression `-2^2` evaluates to -4 because the operator `^` has higher precedence than the unary `-` operator:


``` r
-2^2
#> [1] -4
```

The same can also be seen by its AST:


``` r
ast(-2^2)
#> █─`-` 
#> └─█─`^` 
#>   ├─2 
#>   └─2
```

A less confusing way to write this would be:


``` r
-(2^2)
#> [1] -4
```

**Q4.** What does `!1 + !1` return? Why?

**A3.** The expression `!1 + !1` evaluates to FALSE.

This is because the `!` operator has higher precedence than the unary `+` operator. Thus, `!1` evaluates to `FALSE`, which is added to `1 + FALSE`, which evaluates to `1`, and then logically negated to `!1`, or `FALSE`.

This can be easily seen by its AST:


``` r
ast(!1 + !1)
#> █─`!` 
#> └─█─`+` 
#>   ├─1 
#>   └─█─`!` 
#>     └─1
```

**Q5.** Why does `x1 <- x2 <- x3 <- 0` work? Describe the two reasons.

**A5.** There are two reasons why the following works as expected:


``` r
x1 <- x2 <- x3 <- 0
```

- The `<-` operator is right associative.

Therefore, the order of assignment here is:

```r
(x3 <- 0)
(x2 <- x3)
(x1 <- x2)
```

- The `<-` operator invisibly returns the assigned value.


``` r
(x <- 1)
#> [1] 1
```

This is easy to surmise from its AST:


``` r
ast(x1 <- x2 <- x3 <- 0)
#> █─`<-` 
#> ├─x1 
#> └─█─`<-` 
#>   ├─x2 
#>   └─█─`<-` 
#>     ├─x3 
#>     └─0
```

**Q6.** Compare the ASTs of `x + y %+% z` and `x ^ y %+% z`. What have you learned about the precedence of custom infix functions?

**A6.** Looking at the ASTs for these expressions,


``` r
ast(x + y %+% z)
#> █─`+` 
#> ├─x 
#> └─█─`%+%` 
#>   ├─y 
#>   └─z

ast(x^y %+% z)
#> █─`%+%` 
#> ├─█─`^` 
#> │ ├─x 
#> │ └─y 
#> └─z
```

we can say that the custom infix operator `%+%` has:

- higher precedence than the `+` operator
- lower precedence than the `^` operator

**Q7.** What happens if you call `parse_expr()` with a string that generates multiple expressions? e.g. `parse_expr("x + 1; y + 1")`

**A7.** It produced an error:


``` r
parse_expr("x + 1; y + 1")
#> Error in `parse_expr()`:
#> ! `x` must contain exactly 1 expression, not 2.
```

This is expected based on the docs:

> parse_expr() returns one expression. If the text contains more than one expression (separated by semicolons or new lines), an error is issued. 

We instead need to use `parse_exprs()`:


``` r
parse_exprs("x + 1; y + 1")
#> [[1]]
#> x + 1
#> 
#> [[2]]
#> y + 1
```

**Q8.** What happens if you attempt to parse an invalid expression? e.g. `"a +"` or `"f())"`.

**A8.** An invalid expression produces an error:


``` r
parse_expr("a +")
#> Error in parse(text = x, keep.source = FALSE): <text>:2:0: unexpected end of input
#> 1: a +
#>    ^

parse_expr("f())")
#> Error in parse(text = x, keep.source = FALSE): <text>:1:4: unexpected ')'
#> 1: f())
#>        ^
```

Since the underlying `parse()` function produces an error:


``` r
parse(text = "a +")
#> Error in parse(text = "a +"): <text>:2:0: unexpected end of input
#> 1: a +
#>    ^

parse(text = "f())")
#> Error in parse(text = "f())"): <text>:1:4: unexpected ')'
#> 1: f())
#>        ^
```

**Q9.** `deparse()` produces vectors when the input is long. For example, the following call produces a vector of length two:


``` r
expr <- expr(g(a + b + c + d + e + f + g + h + i + j + k + l +
  m + n + o + p + q + r + s + t + u + v + w + x + y + z))
deparse(expr)
```

What does `expr_text()` do instead?

**A9.** The only difference between `deparse()` and `expr_text()` is that the latter turns the (possibly multi-line) expression into a single string. 


``` r
expr <- expr(g(a + b + c + d + e + f + g + h + i + j + k + l +
  m + n + o + p + q + r + s + t + u + v + w + x + y + z))

deparse(expr)
#> [1] "g(a + b + c + d + e + f + g + h + i + j + k + l + m + n + o + "
#> [2] "    p + q + r + s + t + u + v + w + x + y + z)"

expr_text(expr)
#> [1] "g(a + b + c + d + e + f + g + h + i + j + k + l + m + n + o + \n    p + q + r + s + t + u + v + w + x + y + z)"
```

**Q10.** `pairwise.t.test()` assumes that `deparse()` always returns a length one character vector. Can you construct an input that violates this expectation? What happens?

**A10** Since R 4.0, it is not possible to violate this expectation since the new implementation produces a single string no matter the input:

> New function `deparse1()` produces one string, wrapping `deparse()`, to be used typically in `deparse1(substitute(*))`

## Walking AST with recursive functions (Exercises 18.5.3)

**Q1.** `logical_abbr()` returns `TRUE` for `T(1, 2, 3)`. How could you modify `logical_abbr_rec()` so that it ignores function calls that use `T` or `F`?

**A1.** To avoid function calls that use `T` or `F`, we just need to ignore the function position in call objects:



Let's try it out:


``` r
logical_abbr_rec(expr(T(1, 2, 3)))
#> [1] FALSE

logical_abbr_rec(expr(F(1, 2, 3)))
#> [1] FALSE

logical_abbr_rec(expr(T))
#> [1] TRUE

logical_abbr_rec(expr(F))
#> [1] TRUE
```

**Q2.** `logical_abbr()` works with expressions. It currently fails when you give it a function. Why? How could you modify `logical_abbr()` to make it work? What components of a function will you need to recurse over?


``` r
logical_abbr(function(x = TRUE) {
  g(x + T)
})
```

**A2.** Surprisingly, `logical_abbr()` currently doesn't fail with closures:



To see why, let's see what type of object is produced when we capture user provided closure:


``` r
print_enexpr <- function(.f) {
  print(typeof(enexpr(.f)))
  print(is.call(enexpr(.f)))
}

print_enexpr(function(x = TRUE) {
  g(x + T)
})
#> [1] "language"
#> [1] TRUE
```

Given that closures are converted to `call` objects, it is not a surprise that the function works:


``` r
logical_abbr(function(x = TRUE) {
  g(x + T)
})
#> [1] TRUE
```

The function only fails if it can't find any negative case. For example, instead of returning `FALSE`, this produces an error for reasons that remain (as of yet) elusive to me:

<!-- TODO -->


``` r
logical_abbr(function(x = TRUE) {
  g(x + TRUE)
})
#> [1] FALSE
```

**Q3.** Modify `find_assign` to also detect assignment using replacement functions, i.e. `names(x) <- y`.

**A3.** Although both simple assignment (`x <- y`) and assignment using replacement functions (`names(x) <- y`) have `<-` operator in their call, in the latter case, `names(x)` will be a call object and not a symbol:


``` r
expr1 <- expr(names(x) <- y)
as.list(expr1)
#> [[1]]
#> `<-`
#> 
#> [[2]]
#> names(x)
#> 
#> [[3]]
#> y
typeof(expr1[[2]])
#> [1] "language"

expr2 <- expr(x <- y)
as.list(expr2)
#> [[1]]
#> `<-`
#> 
#> [[2]]
#> x
#> 
#> [[3]]
#> y
typeof(expr2[[2]])
#> [1] "symbol"
```

That's how we can detect this kind of assignment by checking if the second element of the expression is a `symbol` or `language` type object. 


``` r
expr_type <- function(x) {
  if (is_syntactic_literal(x)) {
    "constant"
  } else if (is.symbol(x)) {
    "symbol"
  } else if (is.call(x)) {
    "call"
  } else if (is.pairlist(x)) {
    "pairlist"
  } else {
    typeof(x)
  }
}

switch_expr <- function(x, ...) {
  switch(expr_type(x),
    ...,
    stop("Don't know how to handle type ", typeof(x), call. = FALSE)
  )
}

flat_map_chr <- function(.x, .f, ...) {
  purrr::flatten_chr(purrr::map(.x, .f, ...))
}

extract_symbol <- function(x) {
  if (is_symbol(x[[2]])) {
    as_string(x[[2]])
  } else {
    extract_symbol(as.list(x[[2]]))
  }
}

find_assign_call <- function(x) {
  if (is_call(x, "<-") && is_symbol(x[[2]])) {
    lhs <- as_string(x[[2]])
    children <- as.list(x)[-1]
  } else if (is_call(x, "<-") && is_call(x[[2]])) {
    lhs <- extract_symbol(as.list(x[[2]]))
    children <- as.list(x)[-1]
  } else {
    lhs <- character()
    children <- as.list(x)
  }

  c(lhs, flat_map_chr(children, find_assign_rec))
}

find_assign_rec <- function(x) {
  switch_expr(x,
    # Base cases
    constant = ,
    symbol = character(),

    # Recursive cases
    pairlist = flat_map_chr(x, find_assign_rec),
    call = find_assign_call(x)
  )
}

find_assign <- function(x) find_assign_rec(enexpr(x))
```

Let's try it out:


``` r
find_assign(names(x))
#> character(0)

find_assign(names(x) <- y)
#> [1] "x"

find_assign(names(f(x)) <- y)
#> [1] "x"

find_assign(names(x) <- y <- z <- NULL)
#> [1] "x" "y" "z"

find_assign(a <- b <- c <- 1)
#> [1] "a" "b" "c"

find_assign(system.time(x <- print(y <- 5)))
#> [1] "x" "y"
```

**Q4.** Write a function that extracts all calls to a specified function.

**A4.** Here is a function that extracts all calls to a specified function:


``` r
find_function_call <- function(x, .f) {
  if (is_call(x)) {
    if (is_call(x, .f)) {
      list(x)
    } else {
      purrr::map(as.list(x), ~ find_function_call(.x, .f)) %>%
        purrr::compact() %>%
        unlist(use.names = FALSE)
    }
  }
}

# example-1: with infix operator `:`
find_function_call(expr(mean(1:2)), ":")
#> [[1]]
#> 1:2

find_function_call(expr(sum(mean(1:2))), ":")
#> [[1]]
#> 1:2

find_function_call(expr(list(1:5, 4:6, 3:9)), ":")
#> [[1]]
#> 1:5
#> 
#> [[2]]
#> 4:6
#> 
#> [[3]]
#> 3:9

find_function_call(expr(list(1:5, sum(4:6), mean(3:9))), ":")
#> [[1]]
#> 1:5
#> 
#> [[2]]
#> 4:6
#> 
#> [[3]]
#> 3:9

# example-2: with assignment operator `<-`
find_function_call(expr(names(x)), "<-")
#> NULL

find_function_call(expr(names(x) <- y), "<-")
#> [[1]]
#> names(x) <- y

find_function_call(expr(names(f(x)) <- y), "<-")
#> [[1]]
#> names(f(x)) <- y

find_function_call(expr(names(x) <- y <- z <- NULL), "<-")
#> [[1]]
#> names(x) <- y <- z <- NULL

find_function_call(expr(a <- b <- c <- 1), "<-")
#> [[1]]
#> a <- b <- c <- 1

find_function_call(expr(system.time(x <- print(y <- 5))), "<-")
#> [[1]]
#> x <- print(y <- 5)
```

## Session information


``` r
sessioninfo::session_info(include_base = TRUE)
#> ─ Session info ───────────────────────────────────────────
#>  setting  value
#>  version  R version 4.5.0 (2025-04-11)
#>  os       Ubuntu 24.04.2 LTS
#>  system   x86_64, linux-gnu
#>  ui       X11
#>  language (EN)
#>  collate  C.UTF-8
#>  ctype    C.UTF-8
#>  tz       UTC
#>  date     2025-04-27
#>  pandoc   3.6.4 @ /opt/hostedtoolcache/pandoc/3.6.4/x64/ (via rmarkdown)
#>  quarto   NA
#> 
#> ─ Packages ───────────────────────────────────────────────
#>  package     * version date (UTC) lib source
#>  base        * 4.5.0   2025-04-26 [3] local
#>  bookdown      0.43    2025-04-15 [1] RSPM
#>  cli           3.6.5   2025-04-23 [1] RSPM
#>  compiler      4.5.0   2025-04-26 [3] local
#>  crayon        1.5.3   2024-06-20 [1] RSPM
#>  datasets    * 4.5.0   2025-04-26 [3] local
#>  digest        0.6.37  2024-08-19 [1] RSPM
#>  emoji         16.0.0  2024-10-28 [1] RSPM
#>  evaluate      1.0.3   2025-01-10 [1] RSPM
#>  fastmap       1.2.0   2024-05-15 [1] RSPM
#>  glue          1.8.0   2024-09-30 [1] RSPM
#>  graphics    * 4.5.0   2025-04-26 [3] local
#>  grDevices   * 4.5.0   2025-04-26 [3] local
#>  htmltools     0.5.8.1 2024-04-04 [1] RSPM
#>  knitr         1.50    2025-03-16 [1] RSPM
#>  lifecycle     1.0.4   2023-11-07 [1] RSPM
#>  lobstr      * 1.1.2   2022-06-22 [1] RSPM
#>  magrittr    * 2.0.3   2022-03-30 [1] RSPM
#>  methods     * 4.5.0   2025-04-26 [3] local
#>  pillar        1.10.2  2025-04-05 [1] RSPM
#>  purrr         1.0.4   2025-02-05 [1] RSPM
#>  rlang       * 1.1.6   2025-04-11 [1] RSPM
#>  rmarkdown     2.29    2024-11-04 [1] RSPM
#>  sessioninfo   1.2.3   2025-02-05 [1] RSPM
#>  stats       * 4.5.0   2025-04-26 [3] local
#>  stringi       1.8.7   2025-03-27 [1] RSPM
#>  stringr       1.5.1   2023-11-14 [1] RSPM
#>  tools         4.5.0   2025-04-26 [3] local
#>  utils       * 4.5.0   2025-04-26 [3] local
#>  vctrs         0.6.5   2023-12-01 [1] RSPM
#>  xfun          0.52    2025-04-02 [1] RSPM
#>  yaml          2.3.10  2024-07-26 [1] RSPM
#> 
#>  [1] /home/runner/work/_temp/Library
#>  [2] /opt/R/4.5.0/lib/R/site-library
#>  [3] /opt/R/4.5.0/lib/R/library
#>  * ── Packages attached to the search path.
#> 
#> ──────────────────────────────────────────────────────────
```

