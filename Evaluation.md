# Evaluation



Attaching the needed libraries:


``` r
library(rlang)
```

## Evaluation basics (Exercises 20.2.4)

---

**Q1.** Carefully read the documentation for `source()`. What environment does it use by default? What if you supply `local = TRUE`? How do you provide a custom environment?

**A1.** The parameter `local` for `source()` decides the environment in which the parsed expressions are evaluated. 

By default `local = FALSE`, this corresponds to the user's workspace (the global environment, i.e.). 


``` r
withr::with_tempdir(
  code = {
    f <- tempfile()
    writeLines("rlang::env_print()", f)
    foo <- function() source(f, local = FALSE)
    foo()
  }
)
#> <environment: global>
#> Parent: <environment: package:rlang>
#> Bindings:
#> • .Random.seed: <int>
#> • foo: <fn>
#> • emojis: <chr>
#> • f: <chr>
```

If `local = TRUE`, then the environment from which `source()` is called will be used.


``` r
withr::with_tempdir(
  code = {
    f <- tempfile()
    writeLines("rlang::env_print()", f)
    foo <- function() source(f, local = TRUE)
    foo()
  }
)
#> <environment: 0x55db9aea2e20>
#> Parent: <environment: global>
```

To specify a custom environment, the `sys.source()` function can be used, which provides an `envir` parameter.

---

**Q2.** Predict the results of the following lines of code:


``` r
eval(expr(eval(expr(eval(expr(2 + 2))))))
eval(eval(expr(eval(expr(eval(expr(2 + 2)))))))
expr(eval(expr(eval(expr(eval(expr(2 + 2)))))))
```

**A2.** Correctly predicted 😉


``` r
eval(expr(eval(expr(eval(expr(2 + 2))))))
#> [1] 4

eval(eval(expr(eval(expr(eval(expr(2 + 2)))))))
#> [1] 4

expr(eval(expr(eval(expr(eval(expr(2 + 2)))))))
#> eval(expr(eval(expr(eval(expr(2 + 2))))))
```

---

**Q3.** Fill in the function bodies below to re-implement `get()` using `sym()` and `eval()`, and `assign()` using `sym()`, `expr()`, and `eval()`. Don't worry about the multiple ways of choosing an environment that `get()` and `assign()` support; assume that the user supplies it explicitly.


``` r
# name is a string
get2 <- function(name, env) {}
assign2 <- function(name, value, env) {}
```

**A3.** Here are the required re-implementations:

- `get()`


``` r
get2 <- function(name, env = caller_env()) {
  name <- sym(name)
  eval(name, env)
}

x <- 2

get2("x")
#> [1] 2
get("x")
#> [1] 2

y <- 1:4
assign("y[1]", 2)

get2("y[1]")
#> [1] 2
get("y[1]")
#> [1] 2
```

- `assign()`


``` r
assign2 <- function(name, value, env = caller_env()) {
  name <- sym(name)
  eval(expr(!!name <- !!value), env)
}

assign("y1", 4)
y1
#> [1] 4

assign2("y2", 4)
y2
#> [1] 4
```

---

**Q4.** Modify `source2()` so it returns the result of *every* expression, not just the last one. Can you eliminate the for loop?

**A4.** We can use `purrr::map()` to iterate over every expression and return result of every expression:


``` r
source2 <- function(path, env = caller_env()) {
  file <- paste(readLines(path, warn = FALSE), collapse = "\n")
  exprs <- parse_exprs(file)
  purrr::map(exprs, ~ eval(.x, env))
}

withr::with_tempdir(
  code = {
    f <- tempfile(fileext = ".R")
    writeLines("1 + 1; 2 + 4", f)
    source2(f)
  }
)
#> [[1]]
#> [1] 2
#> 
#> [[2]]
#> [1] 6
```

---

**Q5.** We can make `base::local()` slightly easier to understand by spreading out over multiple lines:


``` r
local3 <- function(expr, envir = new.env()) {
  call <- substitute(eval(quote(expr), envir))
  eval(call, envir = parent.frame())
}
```

Explain how `local()` works in words. (Hint: you might want to `print(call)` to help understand what `substitute()` is doing, and read the documentation to remind yourself what environment `new.env()` will inherit from.)

**A5.** In order to figure out how this function works, let's add the suggested `print(call)`:


``` r
local3 <- function(expr, envir = new.env()) {
  call <- substitute(eval(quote(expr), envir))
  print(call)

  eval(call, envir = parent.frame())
}

local3({
  x <- 10
  y <- 200
  x + y
})
#> eval(quote({
#>     x <- 10
#>     y <- 200
#>     x + y
#> }), new.env())
#> [1] 210
```

As docs for `substitute()` mention:

> Substituting and quoting often cause confusion when the argument is expression(...). The result is a call to the expression constructor function and needs to be evaluated with eval to give the actual expression object.

Thus, to get the actual expression object, quoted expression needs to be evaluated using `eval()`:


``` r
is_expression(eval(quote({
  x <- 10
  y <- 200
  x + y
}), new.env()))
#> [1] TRUE
```

Finally, the generated `call` is evaluated in the caller environment. So the final function call looks like the following:


``` r
# outer environment
eval(
  # inner environment
  eval(quote({
    x <- 10
    y <- 200
    x + y
  }), new.env()),
  envir = parent.frame()
)
```

Note here that the bindings for `x` and `y` are found in the inner environment, while bindings for functions `eval()`, `quote()`, etc. are found in the outer environment. 

---

## Quosures (Exercises 20.3.6)

---

**Q1.** Predict what each of the following quosures will return if evaluated.


``` r
q1 <- new_quosure(expr(x), env(x = 1))
q1
#> <quosure>
#> expr: ^x
#> env:  0x55db9b5d7538
q2 <- new_quosure(expr(x + !!q1), env(x = 10))
q2
#> <quosure>
#> expr: ^x + (^x)
#> env:  0x55db9cae51a0
q3 <- new_quosure(expr(x + !!q2), env(x = 100))
q3
#> <quosure>
#> expr: ^x + (^x + (^x))
#> env:  0x55db9cd53b00
```

**A1.** Correctly predicted 😉


``` r
q1 <- new_quosure(expr(x), env(x = 1))
eval_tidy(q1)
#> [1] 1

q2 <- new_quosure(expr(x + !!q1), env(x = 10))
eval_tidy(q2)
#> [1] 11

q3 <- new_quosure(expr(x + !!q2), env(x = 100))
eval_tidy(q3)
#> [1] 111
```

---

**Q2.** Write an `enenv()` function that captures the environment associated with an argument. (Hint: this should only require two function calls.)

**A2.** We can make use of the `get_env()` helper to get the environment associated with an argument:


``` r
enenv <- function(x) {
  x <- enquo(x)
  get_env(x)
}

enenv(x)
#> <environment: R_GlobalEnv>

foo <- function(x) enenv(x)
foo()
#> <environment: 0x55db9d47c358>
```

---

## Data masks (Exercises 20.4.6)

---

**Q1.** Why did I use a `for` loop in `transform2()` instead of `map()`? Consider `transform2(df, x = x * 2, x = x * 2)`.

**A1.** To see why `map()` is not appropriate for this function, let's create a version of the function with `map()` and see what happens.


``` r
transform2 <- function(.data, ...) {
  dots <- enquos(...)

  for (i in seq_along(dots)) {
    name <- names(dots)[[i]]
    dot <- dots[[i]]

    .data[[name]] <- eval_tidy(dot, .data)
  }

  .data
}

transform3 <- function(.data, ...) {
  dots <- enquos(...)

  purrr::map(dots, function(x, .data = .data) {
    name <- names(x)
    dot <- x

    .data[[name]] <- eval_tidy(dot, .data)

    .data
  })
}
```

When we use a `for()` loop, in each iteration, we are updating the `x` column with the current expression under evaluation. That is, repeatedly modifying the same column works. 


``` r
df <- data.frame(x = 1:3)
transform2(df, x = x * 2, x = x * 2)
#>    x
#> 1  4
#> 2  8
#> 3 12
```

If we use `map()` instead, we are trying to evaluate all expressions at the same time; i.e., the same column is being attempted to modify on using multiple expressions.


``` r
df <- data.frame(x = 1:3)
transform3(df, x = x * 2, x = x * 2)
#> Error in `purrr::map()`:
#> ℹ In index: 1.
#> ℹ With name: x.
#> Caused by error:
#> ! promise already under evaluation: recursive default argument reference or earlier problems?
```

---

**Q2.** Here's an alternative implementation of `subset2()`:


``` r
subset3 <- function(data, rows) {
  rows <- enquo(rows)
  eval_tidy(expr(data[!!rows, , drop = FALSE]), data = data)
}
df <- data.frame(x = 1:3)
subset3(df, x == 1)
```

Compare and contrast `subset3()` to `subset2()`. What are its advantages and disadvantages?

**A2.** Let's first juxtapose these functions and their outputs so that we can compare them better.


``` r
subset2 <- function(data, rows) {
  rows <- enquo(rows)
  rows_val <- eval_tidy(rows, data)
  stopifnot(is.logical(rows_val))

  data[rows_val, , drop = FALSE]
}

df <- data.frame(x = 1:3)
subset2(df, x == 1)
#>   x
#> 1 1
```


``` r
subset3 <- function(data, rows) {
  rows <- enquo(rows)
  eval_tidy(expr(data[!!rows, , drop = FALSE]), data = data)
}

subset3(df, x == 1)
#>   x
#> 1 1
```

**Disadvantages of `subset3()` over `subset2()`**

When the filtering conditions specified in `rows` don't evaluate to a logical, the function doesn't fail informatively. Indeed, it silently returns incorrect result.


``` r
rm("x")
exists("x")
#> [1] FALSE

subset2(df, x + 1)
#> Error in subset2(df, x + 1): is.logical(rows_val) is not TRUE

subset3(df, x + 1)
#>     x
#> 2   2
#> 3   3
#> NA NA
```

**Advantages of `subset3()` over `subset2()`**

Some might argue that the function being shorter is an advantage, but this is very much a subjective preference.

---

**Q3.** The following function implements the basics of `dplyr::arrange()`. Annotate each line with a comment explaining what it does. Can you explain why `!!.na.last` is strictly correct, but omitting the `!!` is unlikely to cause problems?


``` r
arrange2 <- function(.df, ..., .na.last = TRUE) {
  args <- enquos(...)
  order_call <- expr(order(!!!args, na.last = !!.na.last))
  ord <- eval_tidy(order_call, .df)
  stopifnot(length(ord) == nrow(.df))
  .df[ord, , drop = FALSE]
}
```

**A3.** Annotated version of the function:


``` r
arrange2 <- function(.df, ..., .na.last = TRUE) {
  # capture user-supplied expressions (and corresponding environments) as quosures
  args <- enquos(...)

  # create a call object by splicing a list of quosures
  order_call <- expr(order(!!!args, na.last = !!.na.last))

  # and evaluate the constructed call in the data frame
  ord <- eval_tidy(order_call, .df)

  # sanity check
  stopifnot(length(ord) == nrow(.df))

  .df[ord, , drop = FALSE]
}
```

To see why it doesn't matter whether whether we unquote the `.na.last` argument or not, let's have a look at this smaller example:


``` r
x <- TRUE
eval(expr(c(x = !!x)))
#>    x 
#> TRUE
eval(expr(c(x = x)))
#>    x 
#> TRUE
```

As can be seen:

- without unquoting, `.na.last` is found in the function environment
- with unquoting, `.na.last` is included in the `order` call object itself

---

## Using tidy evaluation (Exercises 20.5.4)

---

**Q1.** I've included an alternative implementation of `threshold_var()` below. What makes it different to the approach I used above? What makes it harder?


``` r
threshold_var <- function(df, var, val) {
  var <- ensym(var)
  subset2(df, `$`(.data, !!var) >= !!val)
}
```

**A1.** First, let's compare the two definitions for the same function and make sure that they produce the same output:


``` r
threshold_var_old <- function(df, var, val) {
  var <- as_string(ensym(var))
  subset2(df, .data[[var]] >= !!val)
}

threshold_var_new <- threshold_var

df <- data.frame(x = 1:10)

identical(
  threshold_var(df, x, 8),
  threshold_var(df, x, 8)
)
#> [1] TRUE
```

The key difference is in the subsetting operator used:

- The old version uses non-quoting `[[` operator. Thus, `var` argument first needs to be converted to a string.
- The new version uses quoting `$` operator. Thus, `var` argument is first quoted and then unquoted (using `!!`).

---

## Base evaluation (Exercises 20.6.3)

---

**Q1.** Why does this function fail?


``` r
lm3a <- function(formula, data) {
  formula <- enexpr(formula)
  lm_call <- expr(lm(!!formula, data = data))
  eval(lm_call, caller_env())
}

lm3a(mpg ~ disp, mtcars)$call
#> Error in as.data.frame.default(data, optional = TRUE):
#> cannot coerce class ‘"function"’ to a data.frame
```

**A1.** This doesn't work because when `lm_call` call is evaluated in `caller_env()`, it finds a binding for `base::data()` function, and not `data` from execution environment.

To make it work, we need to unquote `data` into the expression:


``` r
lm3a <- function(formula, data) {
  formula <- enexpr(formula)
  lm_call <- expr(lm(!!formula, data = !!data))
  eval(lm_call, caller_env())
}

is_call(lm3a(mpg ~ disp, mtcars)$call)
#> [1] TRUE
```

---

**Q2.** When model building, typically the response and data are relatively constant while you rapidly experiment with different predictors. Write a small wrapper that allows you to reduce duplication in the code below.


``` r
lm(mpg ~ disp, data = mtcars)
lm(mpg ~ I(1 / disp), data = mtcars)
lm(mpg ~ disp * cyl, data = mtcars)
```

**A2.** Here is a small wrapper that allows you to enter only the predictors:


``` r
lm_custom <- function(data = mtcars, x, y = mpg) {
  x <- enexpr(x)
  y <- enexpr(y)
  data <- enexpr(data)

  lm_call <- expr(lm(formula = !!y ~ !!x, data = !!data))

  eval(lm_call, caller_env())
}

identical(
  lm_custom(x = disp),
  lm(mpg ~ disp, data = mtcars)
)
#> [1] TRUE

identical(
  lm_custom(x = I(1 / disp)),
  lm(mpg ~ I(1 / disp), data = mtcars)
)
#> [1] TRUE

identical(
  lm_custom(x = disp * cyl),
  lm(mpg ~ disp * cyl, data = mtcars)
)
#> [1] TRUE
```

But the function is flexible enough to also allow changing both the data and the dependent variable:


``` r
lm_custom(data = iris, x = Sepal.Length, y = Petal.Width)
#> 
#> Call:
#> lm(formula = Petal.Width ~ Sepal.Length, data = iris)
#> 
#> Coefficients:
#>  (Intercept)  Sepal.Length  
#>      -3.2002        0.7529
```

---

**Q3.** Another way to write `resample_lm()` would be to include the resample expression (`data[sample(nrow(data), replace = TRUE), , drop = FALSE]`) in the data argument. Implement that approach. What are the advantages? What are the disadvantages?

**A3.** In this variant of `resample_lm()`, we are providing the resampled data as an argument. 


``` r
resample_lm3 <- function(formula,
                         data,
                         resample_data = data[sample(nrow(data), replace = TRUE), , drop = FALSE],
                         env = current_env()) {
  formula <- enexpr(formula)
  lm_call <- expr(lm(!!formula, data = resample_data))
  expr_print(lm_call)
  eval(lm_call, env)
}

df <- data.frame(x = 1:10, y = 5 + 3 * (1:10) + round(rnorm(10), 2))
resample_lm3(y ~ x, data = df)
#> lm(y ~ x, data = resample_data)
#> 
#> Call:
#> lm(formula = y ~ x, data = resample_data)
#> 
#> Coefficients:
#> (Intercept)            x  
#>       2.654        3.420
```

This makes use of R's lazy evaluation of function arguments. That is, `resample_data` argument will be evaluated only when it is needed in the function.

---

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
#>  date     2025-04-20
#>  pandoc   3.6.4 @ /opt/hostedtoolcache/pandoc/3.6.4/x64/ (via rmarkdown)
#>  quarto   NA
#> 
#> ─ Packages ───────────────────────────────────────────────
#>  package     * version date (UTC) lib source
#>  base        * 4.5.0   2025-04-11 [3] local
#>  bookdown      0.43    2025-04-15 [1] RSPM
#>  cli           3.6.4   2025-02-13 [1] RSPM
#>  compiler      4.5.0   2025-04-11 [3] local
#>  datasets    * 4.5.0   2025-04-11 [3] local
#>  digest        0.6.37  2024-08-19 [1] RSPM
#>  emoji         16.0.0  2024-10-28 [1] RSPM
#>  evaluate      1.0.3   2025-01-10 [1] RSPM
#>  fastmap       1.2.0   2024-05-15 [1] RSPM
#>  glue          1.8.0   2024-09-30 [1] RSPM
#>  graphics    * 4.5.0   2025-04-11 [3] local
#>  grDevices   * 4.5.0   2025-04-11 [3] local
#>  htmltools     0.5.8.1 2024-04-04 [1] RSPM
#>  knitr         1.50    2025-03-16 [1] RSPM
#>  lifecycle     1.0.4   2023-11-07 [1] RSPM
#>  magrittr    * 2.0.3   2022-03-30 [1] RSPM
#>  methods     * 4.5.0   2025-04-11 [3] local
#>  pillar        1.10.2  2025-04-05 [1] RSPM
#>  purrr         1.0.4   2025-02-05 [1] RSPM
#>  rlang       * 1.1.6   2025-04-11 [1] RSPM
#>  rmarkdown     2.29    2024-11-04 [1] RSPM
#>  sessioninfo   1.2.3   2025-02-05 [1] RSPM
#>  stats       * 4.5.0   2025-04-11 [3] local
#>  stringi       1.8.7   2025-03-27 [1] RSPM
#>  stringr       1.5.1   2023-11-14 [1] RSPM
#>  tools         4.5.0   2025-04-11 [3] local
#>  utils       * 4.5.0   2025-04-11 [3] local
#>  vctrs         0.6.5   2023-12-01 [1] RSPM
#>  withr         3.0.2   2024-10-28 [1] RSPM
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
