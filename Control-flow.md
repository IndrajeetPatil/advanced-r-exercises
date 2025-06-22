# Control flow



## Choices (Exercises 5.2.4)

**Q1.** What type of vector does each of the following calls to `ifelse()` return?


``` r
ifelse(TRUE, 1, "no")
ifelse(FALSE, 1, "no")
ifelse(NA, 1, "no")
```

Read the documentation and write down the rules in your own words.

**A1.** Here are the rules about what a call to `ifelse()` might return: 

- It is type unstable, i.e. the type of return will depend on the type of which condition is true (`yes` or `no`, i.e.): 


``` r
ifelse(TRUE, 1, "no") # `numeric` returned
#> [1] 1
ifelse(FALSE, 1, "no") # `character` returned
#> [1] "no"
```

- It works only for cases where `test` argument evaluates to a `logical` type:


``` r
ifelse(NA_real_, 1, "no")
#> [1] NA
ifelse(NaN, 1, "no")
#> [1] NA
```

- If `test` is argument is of logical type, but `NA`, it will return `NA`:


``` r
ifelse(NA, 1, "no")
#> [1] NA
```

- If the `test` argument doesn't resolve to `logical` type, it will try to coerce the output to a `logical` type:


``` r
# will work
ifelse("TRUE", 1, "no")
#> [1] 1
ifelse("false", 1, "no")
#> [1] "no"

# won't work
ifelse("tRuE", 1, "no")
#> [1] NA
ifelse(NaN, 1, "no")
#> [1] NA
```

This is also clarified in the docs for this function:

> A vector of the same length and attributes (including dimensions and `"class"`) as `test` and data values from the values of `yes` or `no`. The mode of the answer will be coerced from logical to accommodate first any values taken from yes and then any values taken from `no`.

**Q2.** Why does the following code work?


``` r
x <- 1:10
if (length(x)) "not empty" else "empty"
#> [1] "not empty"

x <- numeric()
if (length(x)) "not empty" else "empty"
#> [1] "empty"
```

**A2.** The code works because the conditional expressions in `if()` - even though of `numeric` type - can be successfully coerced to a `logical` type.


``` r
as.logical(length(1:10))
#> [1] TRUE

as.logical(length(numeric()))
#> [1] FALSE
```

## Loops (Exercises 5.3.3)

**Q1.** Why does this code succeed without errors or warnings? 
    

``` r
x <- numeric()
out <- vector("list", length(x))
for (i in 1:length(x)) {
  out[i] <- x[i]^2
}
out
```

**A1.** This works because `1:length(x)` works in both positive and negative directions.


``` r
1:2
#> [1] 1 2
1:0
#> [1] 1 0
1:-3
#> [1]  1  0 -1 -2 -3
```

In this case, since `x` is of length `0`, `i` will go from `1` to `0`. 

Additionally, since out-of-bound (OOB) value for atomic vectors is `NA`, all related operations with OOB values will also produce `NA`.


``` r
x <- numeric()
out <- vector("list", length(x))

for (i in 1:length(x)) {
  print(paste("i:", i, ", x[i]:", x[i], ", out[i]:", out[i]))

  out[i] <- x[i]^2
}
#> [1] "i: 1 , x[i]: NA , out[i]: NULL"
#> [1] "i: 0 , x[i]:  , out[i]: "

out
#> [[1]]
#> [1] NA
```

A way to do avoid this unintended behavior is to use `seq_along()` instead:


``` r
x <- numeric()
out <- vector("list", length(x))

for (i in seq_along(x)) {
  out[i] <- x[i]^2
}

out
#> list()
```

**Q2.** When the following code is evaluated, what can you say about the vector being iterated?


``` r
xs <- c(1, 2, 3)
for (x in xs) {
  xs <- c(xs, x * 2)
}
xs
#> [1] 1 2 3 2 4 6
```

**A2.** The iterator variable `x` initially takes all values of the vector `xs`. We can check this by printing `x` for each iteration:


``` r
xs <- c(1, 2, 3)
for (x in xs) {
  cat("x:", x, "\n")
  xs <- c(xs, x * 2)
  cat("xs:", paste(xs), "\n")
}
#> x: 1 
#> xs: 1 2 3 2 
#> x: 2 
#> xs: 1 2 3 2 4 
#> x: 3 
#> xs: 1 2 3 2 4 6
```

It is worth noting that `x` is not updated *after* each iteration; otherwise, it will take increasingly bigger values of `xs`, and the loop will never end executing.

**Q3.** What does the following code tell you about when the index is updated?


``` r
for (i in 1:3) {
  i <- i * 2
  print(i)
}
#> [1] 2
#> [1] 4
#> [1] 6
```

**A3.** In a `for()` loop the index is updated in the **beginning** of each iteration. Otherwise, we will encounter an infinite loop.


``` r
for (i in 1:3) {
  cat("before: ", i, "\n")
  i <- i * 2
  cat("after:  ", i, "\n")
}
#> before:  1 
#> after:   2 
#> before:  2 
#> after:   4 
#> before:  3 
#> after:   6
```

Also, worth contrasting the behavior of `for()` loop with that of `while()` loop:


``` r
i <- 1
while (i < 4) {
  cat("before: ", i, "\n")
  i <- i * 2
  cat("after:  ", i, "\n")
}
#> before:  1 
#> after:   2 
#> before:  2 
#> after:   4
```

## Session information


``` r
sessioninfo::session_info(include_base = TRUE)
#> ─ Session info ───────────────────────────────────────────
#>  setting  value
#>  version  R version 4.5.1 (2025-06-13)
#>  os       Ubuntu 24.04.2 LTS
#>  system   x86_64, linux-gnu
#>  ui       X11
#>  language (EN)
#>  collate  C.UTF-8
#>  ctype    C.UTF-8
#>  tz       UTC
#>  date     2025-06-22
#>  pandoc   3.7.0.2 @ /opt/hostedtoolcache/pandoc/3.7.0.2/x64/ (via rmarkdown)
#>  quarto   NA
#> 
#> ─ Packages ───────────────────────────────────────────────
#>  package     * version date (UTC) lib source
#>  base        * 4.5.1   2025-06-13 [3] local
#>  bookdown      0.43    2025-04-15 [1] RSPM
#>  cli           3.6.5   2025-04-23 [1] RSPM
#>  compiler      4.5.1   2025-06-13 [3] local
#>  datasets    * 4.5.1   2025-06-13 [3] local
#>  digest        0.6.37  2024-08-19 [1] RSPM
#>  emoji         16.0.0  2024-10-28 [1] RSPM
#>  evaluate      1.0.4   2025-06-18 [1] RSPM
#>  fastmap       1.2.0   2024-05-15 [1] RSPM
#>  glue          1.8.0   2024-09-30 [1] RSPM
#>  graphics    * 4.5.1   2025-06-13 [3] local
#>  grDevices   * 4.5.1   2025-06-13 [3] local
#>  htmltools     0.5.8.1 2024-04-04 [1] RSPM
#>  knitr         1.50    2025-03-16 [1] RSPM
#>  lifecycle     1.0.4   2023-11-07 [1] RSPM
#>  magrittr    * 2.0.3   2022-03-30 [1] RSPM
#>  methods     * 4.5.1   2025-06-13 [3] local
#>  rlang         1.1.6   2025-04-11 [1] RSPM
#>  rmarkdown     2.29    2024-11-04 [1] RSPM
#>  sessioninfo   1.2.3   2025-02-05 [1] RSPM
#>  stats       * 4.5.1   2025-06-13 [3] local
#>  stringi       1.8.7   2025-03-27 [1] RSPM
#>  stringr       1.5.1   2023-11-14 [1] RSPM
#>  tools         4.5.1   2025-06-13 [3] local
#>  utils       * 4.5.1   2025-06-13 [3] local
#>  xfun          0.52    2025-04-02 [1] RSPM
#>  yaml          2.3.10  2024-07-26 [1] RSPM
#> 
#>  [1] /home/runner/work/_temp/Library
#>  [2] /opt/R/4.5.1/lib/R/site-library
#>  [3] /opt/R/4.5.1/lib/R/library
#>  * ── Packages attached to the search path.
#> 
#> ──────────────────────────────────────────────────────────
```
