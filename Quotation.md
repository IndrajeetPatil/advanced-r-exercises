# Quasiquotation



Attaching the needed libraries:


``` r
library(rlang)
library(purrr)
library(lobstr)
library(dplyr)
library(ggplot2)
```

## Motivation (Exercises 19.2.2)

---

**Q1.** For each function in the following base R code, identify which arguments are quoted and which are evaluated.


``` r
library(MASS)

mtcars2 <- subset(mtcars, cyl == 4)

with(mtcars2, sum(vs))
sum(mtcars2$am)

rm(mtcars2)
```

**A1.** To identify which arguments are quoted and which are evaluated, we can use the trick mentioned in the book:

> If you’re ever unsure about whether an argument is quoted or evaluated, try executing the code outside of the function. If it doesn’t work or does something different, then that argument is quoted.

- `library(MASS)`

The `package` argument in `library()` is quoted:


``` r
library(MASS)

MASS
#> Error: object 'MASS' not found
```

- `subset(mtcars, cyl == 4)`

The argument `x` is evaluated, while the argument `subset` is quoted.


``` r
mtcars2 <- subset(mtcars, cyl == 4)

invisible(mtcars)

cyl == 4
#> Error: object 'cyl' not found
```

- `with(mtcars2, sum(vs))`

The argument `data` is evaluated, while `expr` argument is quoted.


``` r
with(mtcars2, sum(vs))
#> [1] 10

invisible(mtcars2)

sum(vs)
#> Error: object 'vs' not found
```

- `sum(mtcars2$am)`

The argument `...` is evaluated.


``` r
sum(mtcars2$am)
#> [1] 8

mtcars2$am
#>  [1] 1 0 0 1 1 1 0 1 1 1 1
```

- `rm(mtcars2)`

The trick we are using so far won't work here since trying to print `mtcars2` will always fail after `rm()` has made a pass at it.


``` r
rm(mtcars2)
```

We can instead look at the docs for `...`:

> ... the objects to be removed, as names (unquoted) or character strings (quoted).

Thus, this argument is not evaluated, but rather quoted.

---

**Q2.** For each function in the following tidyverse code, identify which arguments are quoted and which are evaluated.


``` r
library(dplyr)
library(ggplot2)

by_cyl <- mtcars %>%
  group_by(cyl) %>%
  summarise(mean = mean(mpg))

ggplot(by_cyl, aes(cyl, mean)) +
  geom_point()
```

**A2.** As seen in the answer for **Q1.**, `library()` quotes its first argument:


``` r
library(dplyr)
library(ggplot2)
```

In the following code:

- `%>%` (lazily) evaluates its argument
- `group_by()` and `summarise()` quote their arguments


``` r
by_cyl <- mtcars %>%
  group_by(cyl) %>%
  summarise(mean = mean(mpg))
```

In the following code:

- `ggplot()` evaluates the `data` argument
- `aes()` quotes its arguments


``` r
ggplot(by_cyl, aes(cyl, mean)) +
  geom_point()
```

<img src="Quotation_files/figure-epub3/Quotation-12-1.png" width="100%" />

---

## Quoting (Exercises 19.3.6)

---

**Q1.** How is `expr()` implemented? Look at its source code.

**A1.** Looking at the source code, we can see that `expr()` is a simple wrapper around `enexpr()`, and captures and returns the user-entered expressions:


``` r
rlang::expr
#> function (expr) 
#> {
#>     enexpr(expr)
#> }
#> <bytecode: 0x5621069782f8>
#> <environment: namespace:rlang>
```

For example:


``` r
x <- expr(x <- 1)
x
#> x <- 1
```

In its turn, `enexpr()` calls native code:


``` r
rlang::enexpr
#> function (arg) 
#> {
#>     .Call(ffi_enexpr, substitute(arg), parent.frame())
#> }
#> <bytecode: 0x56210288c5b8>
#> <environment: namespace:rlang>
```


---

**Q2.** Compare and contrast the following two functions. Can you predict the output before running them?


``` r
f1 <- function(x, y) {
  exprs(x = x, y = y)
}
f2 <- function(x, y) {
  enexprs(x = x, y = y)
}
f1(a + b, c + d)
f2(a + b, c + d)
```

**A2.** The `exprs()` captures and returns the expressions specified by the developer instead of their values:


``` r
f1 <- function(x, y) {
  exprs(x = x, y = y)
}

f1(a + b, c + d)
#> $x
#> x
#> 
#> $y
#> y
```

On the other hand, `enexprs()` captures the user-entered expressions and returns their values:


``` r
f2 <- function(x, y) {
  enexprs(x = x, y = y)
}

f2(a + b, c + d)
#> $x
#> a + b
#> 
#> $y
#> c + d
```

---

**Q3.** What happens if you try to use `enexpr()` with an expression (i.e. `enexpr(x + y)`? What happens if `enexpr()` is passed a missing argument?

**A3.** If you try to use `enexpr()` with an expression, it fails because it works only with `symbol`.


``` r
enexpr(x + y)
#> Error in `enexpr()`:
#> ! `arg` must be a symbol
```

If `enexpr()` is passed a missing argument, it returns a missing argument:


``` r
arg <- missing_arg()

enexpr(arg)

is_missing(enexpr(arg))
#> [1] TRUE
```

---

**Q4.** How are `exprs(a)` and `exprs(a = )` different? Think about both the input and the output.

**A4.** The key difference between `exprs(a)` and `exprs(a = )` is that the former will return an unnamed list, while the latter will return a named list. This is because the former is interpreted as an unnamed argument, while the latter a named argument.


``` r
exprs(a)
#> [[1]]
#> a

exprs(a = )
#> $a
```

In both cases, `a` is treated as a symbol:


``` r
map_lgl(exprs(a), is_symbol)
#>      
#> TRUE

map_lgl(exprs(a = ), is_symbol)
#>    a 
#> TRUE
```

But, the argument is missing only in the latter case, since only the name but no corresponding value is provided:


``` r
map_lgl(exprs(a), is_missing)
#>       
#> FALSE

map_lgl(exprs(a = ), is_missing)
#>    a 
#> TRUE
```

---

**Q5.** What are other differences between `exprs()` and `alist()`? Read the  documentation for the named arguments of `exprs()` to find out.

**A5.** Here are some additional differences between `exprs()` and `alist()`.

- Names: If the inputs are not named, `exprs()` provides a way to name them automatically using `.named` argument.


``` r
alist("x" = 1, TRUE, "z" = expr(x + y))
#> $x
#> [1] 1
#> 
#> [[2]]
#> [1] TRUE
#> 
#> $z
#> expr(x + y)

exprs("x" = 1, TRUE, "z" = expr(x + y), .named = TRUE)
#> $x
#> [1] 1
#> 
#> $`TRUE`
#> [1] TRUE
#> 
#> $z
#> expr(x + y)
```

- Ignoring empty arguments: The `.ignore_empty` argument in `exprs()` gives you a much finer control over what to do with the empty arguments, while `alist()` doesn't provide a way to ignore such arguments.


``` r
alist("x" = 1, , TRUE, )
#> $x
#> [1] 1
#> 
#> [[2]]
#> 
#> 
#> [[3]]
#> [1] TRUE
#> 
#> [[4]]

exprs("x" = 1, , TRUE, , .ignore_empty = "trailing")
#> $x
#> [1] 1
#> 
#> [[2]]
#> 
#> 
#> [[3]]
#> [1] TRUE

exprs("x" = 1, , TRUE, , .ignore_empty = "none")
#> $x
#> [1] 1
#> 
#> [[2]]
#> 
#> 
#> [[3]]
#> [1] TRUE
#> 
#> [[4]]

exprs("x" = 1, , TRUE, , .ignore_empty = "all")
#> $x
#> [1] 1
#> 
#> [[2]]
#> [1] TRUE
```

- Names injection: Using `.unquote_names` argument in `exprs()`, we can inject a name for the argument.


``` r
alist(foo := bar)
#> [[1]]
#> `:=`(foo, bar)

exprs(foo := bar, .unquote_names = FALSE)
#> [[1]]
#> `:=`(foo, bar)

exprs(foo := bar, .unquote_names = TRUE)
#> $foo
#> bar
```

---

**Q6.** The documentation for `substitute()` says:

> Substitution takes place by examining each component of the parse tree 
> as follows: 
> 
> * If it is not a bound symbol in `env`, it is unchanged. 
> * If it is a promise object (i.e., a formal argument to a function) 
>   the expression slot of the promise replaces the symbol. 
> * If it is an ordinary variable, its value is substituted, unless 
> `env` is .GlobalEnv in which case the symbol is left unchanged.
  
Create examples that illustrate each of the above cases.

**A6.** See below examples that illustrate each of the above-mentioned cases.

> If it is not a bound symbol in `env`, it is unchanged. 

Symbol `x` is not bound in `env`, so it remains unchanged. 


``` r
substitute(x + y, env = list(y = 2))
#> x + 2
```

> If it is a promise object (i.e., a formal argument to a function) 
> the expression slot of the promise replaces the symbol. 


``` r
msg <- "old"
delayedAssign("myVar", msg) # creates a promise
substitute(myVar)
#> myVar
msg <- "new!"
myVar
#> [1] "new!"
```

> If it is an ordinary variable, its value is substituted, unless 
> `env` is .GlobalEnv in which case the symbol is left unchanged.


``` r
substitute(x + y, env = env(x = 2, y = 1))
#> 2 + 1

x <- 2
y <- 1
substitute(x + y, env = .GlobalEnv)
#> x + y
```

---

## Unquoting (Exercises 19.4.8)

---

**Q1.** Given the following components:


``` r
xy <- expr(x + y)
xz <- expr(x + z)
yz <- expr(y + z)
abc <- exprs(a, b, c)
```

Use quasiquotation to construct the following calls:


``` r
(x + y) / (y + z)
-(x + z)^(y + z)
(x + y) + (y + z) - (x + y)
atan2(x + y, y + z)
sum(x + y, x + y, y + z)
sum(a, b, c)
mean(c(a, b, c), na.rm = TRUE)
foo(a = x + y, b = y + z)
```

**A1.** Using quasiquotation to construct the specified calls:


``` r
xy <- expr(x + y)
xz <- expr(x + z)
yz <- expr(y + z)
abc <- exprs(a, b, c)

expr((!!xy) / (!!yz))
#> (x + y)/(y + z)

expr(-(!!xz)^(!!yz))
#> -(x + z)^(y + z)

expr(((!!xy)) + (!!yz) - (!!xy))
#> (x + y) + (y + z) - (x + y)

call2("atan2", expr(!!xy), expr(!!yz))
#> atan2(x + y, y + z)

call2("sum", expr(!!xy), expr(!!xy), expr(!!yz))
#> sum(x + y, x + y, y + z)

call2("sum", !!!abc)
#> sum(a, b, c)

expr(mean(c(!!!abc), na.rm = TRUE))
#> mean(c(a, b, c), na.rm = TRUE)

call2("foo", a = expr(!!xy), b = expr(!!yz))
#> foo(a = x + y, b = y + z)
```

---

**Q2.** The following two calls print the same, but are actually different:


``` r
(a <- expr(mean(1:10)))
#> mean(1:10)
(b <- expr(mean(!!(1:10))))
#> mean(1:10)
identical(a, b)
#> [1] FALSE
```

What's the difference? Which one is more natural?

**A2.** We can see the difference between these two expression if we convert them to lists:


``` r
as.list(expr(mean(1:10)))
#> [[1]]
#> mean
#> 
#> [[2]]
#> 1:10

as.list(expr(mean(!!(1:10))))
#> [[1]]
#> mean
#> 
#> [[2]]
#>  [1]  1  2  3  4  5  6  7  8  9 10
```

As can be seen, the second element of `a` is a `call` object, while that in `b` is an integer vector:


``` r
waldo::compare(a, b)
#> `old[[2]]` is a call
#> `new[[2]]` is an integer vector (1, 2, 3, 4, 5, ...)
```

The same can also be noticed in ASTs for these expressions:


``` r
ast(expr(mean(1:10)))
#> █─expr 
#> └─█─mean 
#>   └─█─`:` 
#>     ├─1 
#>     └─10

ast(expr(mean(!!(1:10))))
#> █─expr 
#> └─█─mean 
#>   └─<inline integer>
```

The first call is more natural, since the second one inlines a vector directly into the call, something that is rarely done.

---

## `...` (dot-dot-dot) (Exercises 19.6.5)

---

**Q1.** One way to implement `exec()` is shown below. Describe how it works. What are the key ideas?


``` r
exec <- function(f, ..., .env = caller_env()) {
  args <- list2(...)
  do.call(f, args, envir = .env)
}
```

**A1.** The keys ideas that underlie this implementation of `exec()` function are the following:

- It constructs a call using function `f` and its argument `...`, and evaluates the call in the environment `.env`.

- It uses [dynamic dots](https://rlang.r-lib.org/reference/dyn-dots.html) via `list2()`, which means that you can splice arguments using `!!!`, you can inject names using `:=`, and trailing commas are not a problem.

Here is an example:


``` r
vec <- c(1:5, NA)
args_list <- list(trim = 0, na.rm = TRUE)

exec(mean, vec, !!!args_list, , .env = caller_env())
#> [1] 3

rm("exec")
```

---

**Q2.** Carefully read the source code for `interaction()`, `expand.grid()`, and `par()`. Compare and contrast the techniques they use for switching between dots and list behaviour.

**A2.** Source code reveals the following comparison table:

| Function        | Capture the dots    | Handle list input                                                    |
| :-------------- | :------------------ | :------------------------------------------------------------------- |
| `interaction()` | `args <- list(...)` | `length(args) == 1L && is.list(args[[1L]])`                          |
| `expand.grid()` | `args <- list(...)` | `length(args) == 1L && is.list(args[[1L]])`                          |
| `par()`         | `args <- list(...)` | `length(args) == 1L && (is.list(args[[1L]] || is.null(args[[1L]])))` |

All functions capture the dots in a list.

Using these dots, the functions check:

  - if a list was entered as an argument by checking the number of arguments
  - if the count is 1, by checking if the argument is a list

---

**Q3.** Explain the problem with this definition of `set_attr()`


``` r
set_attr <- function(x, ...) {
  attr <- rlang::list2(...)
  attributes(x) <- attr
  x
}
set_attr(1:10, x = 10)
#> Error in attributes(x) <- attr: attributes must be named
```

**A3.** The `set_attr()` function signature has a parameter called `x`, and additionally it uses dynamic dots to pass multiple arguments to specify additional attributes for `x`.

But, as shown in the example, this creates a problem when the attribute is itself named `x`. Naming the arguments won't help either:


``` r
set_attr <- function(x, ...) {
  attr <- rlang::list2(...)
  attributes(x) <- attr
  x
}
set_attr(x = 1:10, x = 10)
#> Error in set_attr(x = 1:10, x = 10): formal argument "x" matched by multiple actual arguments
```

We can avoid these issues by renaming the parameter:


``` r
set_attr <- function(.x, ...) {
  attr <- rlang::list2(...)
  attributes(.x) <- attr
  .x
}

set_attr(.x = 1:10, x = 10)
#>  [1]  1  2  3  4  5  6  7  8  9 10
#> attr(,"x")
#> [1] 10
```

---

## Case studies (Exercises 19.7.5)

---

**Q1.** In the linear-model example, we could replace the `expr()` in `reduce(summands, ~ expr(!!.x + !!.y))` with `call2()`: `reduce(summands, call2, "+")`. Compare and contrast the two approaches. Which do you think is easier to read?

**A1.** We can rewrite the `linear()` function from this chapter using `call2()` as follows:


``` r
linear <- function(var, val) {
  var <- ensym(var)
  coef_name <- map(seq_along(val[-1]), ~ expr((!!var)[[!!.x]]))

  summands <- map2(val[-1], coef_name, ~ expr((!!.x * !!.y)))
  summands <- c(val[[1]], summands)

  reduce(summands, ~ call2("+", .x, .y))
}

linear(x, c(10, 5, -4))
#> 10 + (5 * x[[1L]]) + (-4 * x[[2L]])
```

I personally find the version with `call2()` to be much more readable since the `!!` syntax is a bit esoteric.

---

**Q2.** Re-implement the Box-Cox transform defined below using unquoting and `new_function()`:


``` r
bc <- function(lambda) {
  if (lambda == 0) {
    function(x) log(x)
  } else {
    function(x) (x^lambda - 1) / lambda
  }
}
```

**A2.** Re-implementation of the Box-Cox transform using unquoting and `new_function()`:


``` r
bc_new <- function(lambda) {
  lambda <- enexpr(lambda)

  if (!!lambda == 0) {
    new_function(
      exprs(x = ),
      expr(log(x))
    )
  } else {
    new_function(
      exprs(x = ),
      expr((x^(!!lambda) - 1) / (!!lambda))
    )
  }
}
```

Let's try it out to see if it produces the same output as before:


``` r
bc(0)(1)
#> [1] 0
bc_new(0)(1)
#> [1] 0

bc(2)(2)
#> [1] 1.5
bc_new(2)(2)
#> [1] 1.5
```

---

**Q3.**  Re-implement the simple `compose()` defined below using quasiquotation and `new_function()`:


``` r
compose <- function(f, g) {
  function(...) f(g(...))
}
```

**A3.** Following is a re-implementation of `compose()` using quasiquotation and `new_function()`:


``` r
compose_new <- function(f, g) {
  f <- enexpr(f) # or ensym(f)
  g <- enexpr(g) # or ensym(g)

  new_function(
    exprs(... = ),
    expr((!!f)((!!g)(...)))
  )
}
```

Checking that the new version behaves the same way as the original version:


``` r
not_null <- compose(`!`, is.null)
not_null(4)
#> [1] TRUE

not_null2 <- compose_new(`!`, is.null)
not_null2(4)
#> [1] TRUE
```

---

## Session information


``` r
sessioninfo::session_info(include_base = TRUE)
#> ─ Session info ───────────────────────────────────────────
#>  setting  value
#>  version  R version 4.4.3 (2025-02-28)
#>  os       Ubuntu 24.04.2 LTS
#>  system   x86_64, linux-gnu
#>  ui       X11
#>  language (EN)
#>  collate  C.UTF-8
#>  ctype    C.UTF-8
#>  tz       UTC
#>  date     2025-03-16
#>  pandoc   3.6.3 @ /opt/hostedtoolcache/pandoc/3.6.3/x64/ (via rmarkdown)
#>  quarto   NA
#> 
#> ─ Packages ───────────────────────────────────────────────
#>  package     * version date (UTC) lib source
#>  base        * 4.4.3   2025-02-28 [3] local
#>  bookdown      0.42    2025-01-07 [1] RSPM
#>  cli           3.6.4   2025-02-13 [1] RSPM
#>  colorspace    2.1-1   2024-07-26 [1] RSPM
#>  compiler      4.4.3   2025-02-28 [3] local
#>  crayon        1.5.3   2024-06-20 [1] RSPM
#>  datasets    * 4.4.3   2025-02-28 [3] local
#>  diffobj       0.3.5   2021-10-05 [1] RSPM
#>  digest        0.6.37  2024-08-19 [1] RSPM
#>  dplyr       * 1.1.4   2023-11-17 [1] RSPM
#>  emoji         16.0.0  2024-10-28 [1] RSPM
#>  evaluate      1.0.3   2025-01-10 [1] RSPM
#>  farver        2.1.2   2024-05-13 [1] RSPM
#>  fastmap       1.2.0   2024-05-15 [1] RSPM
#>  generics      0.1.3   2022-07-05 [1] RSPM
#>  ggplot2     * 3.5.1   2024-04-23 [1] RSPM
#>  glue          1.8.0   2024-09-30 [1] RSPM
#>  graphics    * 4.4.3   2025-02-28 [3] local
#>  grDevices   * 4.4.3   2025-02-28 [3] local
#>  grid          4.4.3   2025-02-28 [3] local
#>  gtable        0.3.6   2024-10-25 [1] RSPM
#>  htmltools     0.5.8.1 2024-04-04 [1] RSPM
#>  knitr         1.49    2024-11-08 [1] RSPM
#>  labeling      0.4.3   2023-08-29 [1] RSPM
#>  lifecycle     1.0.4   2023-11-07 [1] RSPM
#>  lobstr      * 1.1.2   2022-06-22 [1] RSPM
#>  magrittr    * 2.0.3   2022-03-30 [1] RSPM
#>  MASS        * 7.3-64  2025-01-04 [3] CRAN (R 4.4.3)
#>  methods     * 4.4.3   2025-02-28 [3] local
#>  munsell       0.5.1   2024-04-01 [1] RSPM
#>  pillar        1.10.1  2025-01-07 [1] RSPM
#>  pkgconfig     2.0.3   2019-09-22 [1] RSPM
#>  purrr       * 1.0.4   2025-02-05 [1] RSPM
#>  R6            2.6.1   2025-02-15 [1] RSPM
#>  rlang       * 1.1.5   2025-01-17 [1] RSPM
#>  rmarkdown     2.29    2024-11-04 [1] RSPM
#>  scales        1.3.0   2023-11-28 [1] RSPM
#>  sessioninfo   1.2.3   2025-02-05 [1] RSPM
#>  stats       * 4.4.3   2025-02-28 [3] local
#>  stringi       1.8.4   2024-05-06 [1] RSPM
#>  stringr       1.5.1   2023-11-14 [1] RSPM
#>  tibble        3.2.1   2023-03-20 [1] RSPM
#>  tidyselect    1.2.1   2024-03-11 [1] RSPM
#>  tools         4.4.3   2025-02-28 [3] local
#>  utils       * 4.4.3   2025-02-28 [3] local
#>  vctrs         0.6.5   2023-12-01 [1] RSPM
#>  waldo         0.6.1   2024-11-07 [1] RSPM
#>  withr         3.0.2   2024-10-28 [1] RSPM
#>  xfun          0.51    2025-02-19 [1] RSPM
#>  yaml          2.3.10  2024-07-26 [1] RSPM
#> 
#>  [1] /home/runner/work/_temp/Library
#>  [2] /opt/R/4.4.3/lib/R/site-library
#>  [3] /opt/R/4.4.3/lib/R/library
#>  * ── Packages attached to the search path.
#> 
#> ──────────────────────────────────────────────────────────
```
