# Names and values



Loading the needed libraries:


``` r
library(lobstr)
```

## Binding basics (Exercise 2.2.2)

---

**Q1.** Explain the relationship between `a`, `b`, `c` and `d` in the following code:


``` r
a <- 1:10
b <- a
c <- b
d <- 1:10
```

**A1.** The names (`a`, `b`, and `c`) have same values and point to the same object in memory, as can be seen by their identical memory addresses:


``` r
obj_addrs <- obj_addrs(list(a, b, c))
unique(obj_addrs)
#> [1] "0x55dfcdb5d760"
```

Except `d`, which is a different object, even if it has the same value as `a`, `b`, and `c`:


``` r
obj_addr(d)
#> [1] "0x55dfcdbe0078"
```

---

**Q2.** The following code accesses the mean function in multiple ways. Do they all point to the same underlying function object? Verify this with `lobstr::obj_addr()`.


``` r
mean
base::mean
get("mean")
evalq(mean)
match.fun("mean")
```

**A2.** All listed function calls point to the same underlying function object in memory, as shown by this object's memory address:


``` r
obj_addrs <- obj_addrs(list(
  mean,
  base::mean,
  get("mean"),
  evalq(mean),
  match.fun("mean")
))

unique(obj_addrs)
#> [1] "0x55dfcb942770"
```

---

**Q3.** By default, base R data import functions, like `read.csv()`, will automatically convert non-syntactic names to syntactic ones. Why might this be problematic? What option allows you to suppress this behaviour?

**A3.** The conversion of non-syntactic names to syntactic ones can sometimes corrupt the data. Some datasets may require non-syntactic names.

To suppress this behavior, one can set `check.names = FALSE`.

---

**Q4.** What rules does `make.names()` use to convert non-syntactic names into syntactic ones?

**A4.** `make.names()` uses following rules to convert non-syntactic names into syntactic ones:

- it prepends non-syntactic names with `X`
- it converts invalid characters (like `@`) to `.`
- it adds a `.` as a suffix if the name is a [reserved keyword](https://stat.ethz.ch/R-manual/R-devel/library/base/html/Reserved.html)


``` r
make.names(c("123abc", "@me", "_yu", "  gh", "else"))
#> [1] "X123abc" "X.me"    "X_yu"    "X..gh"   "else."
```

---

**Q5.** I slightly simplified the rules that govern syntactic names. Why is `.123e1` not a syntactic name? Read `?make.names` for the full details.

**A5.** `.123e1` is not a syntacti name because it is parsed as a number, and not as a string:


``` r
typeof(.123e1)
#> [1] "double"
```

And as the docs mention (emphasis mine):

> A syntactically valid name consists of letters, numbers and the dot or underline characters and starts with a letter or **the dot not followed by a number**.

---

## Copy-on-modify (Exercise 2.3.6)

---

**Q1.** Why is `tracemem(1:10)` not useful?

**A1.** `tracemem()` traces copying of objects in R. For example:


``` r
x <- 1:10

tracemem(x)
#> [1] "<0x55dfccfa0048>"

x <- x + 1

untracemem(x)
```

But since the object created in memory by `1:10` is not assigned a name, it can't be addressed or modified from R, and so there is nothing to trace. 


``` r
obj_addr(1:10)
#> [1] "0x55dfcd793798"

tracemem(1:10)
#> [1] "<0x55dfcd7e0a20>"
```

---

**Q2.** Explain why `tracemem()` shows two copies when you run this code. Hint: carefully look at the difference between this code and the code shown earlier in the section.


``` r
x <- c(1L, 2L, 3L)
tracemem(x)

x[[3]] <- 4
untracemem(x)
```

**A2.** This is because the initial atomic vector is of type `integer`, but `4` (and not `4L`) is of type `double`. This is why a new copy is created.


``` r
x <- c(1L, 2L, 3L)
typeof(x)
#> [1] "integer"
tracemem(x)
#> [1] "<0x55dfce67cdc8>"

x[[3]] <- 4
#> tracemem[0x55dfce67cdc8 -> 0x55dfce70c9e8]: eval eval withVisible withCallingHandlers eval eval with_handlers doWithOneRestart withOneRestart withRestartList doWithOneRestart withOneRestart withRestartList withRestarts <Anonymous> evaluate in_dir in_input_dir eng_r block_exec call_block process_group withCallingHandlers <Anonymous> process_file <Anonymous> <Anonymous> do.call eval eval eval eval eval.parent local 
#> tracemem[0x55dfce70c9e8 -> 0x55dfce78c358]: eval eval withVisible withCallingHandlers eval eval with_handlers doWithOneRestart withOneRestart withRestartList doWithOneRestart withOneRestart withRestartList withRestarts <Anonymous> evaluate in_dir in_input_dir eng_r block_exec call_block process_group withCallingHandlers <Anonymous> process_file <Anonymous> <Anonymous> do.call eval eval eval eval eval.parent local
untracemem(x)

typeof(x)
#> [1] "double"
```

Trying with an integer should not create another copy:


``` r
x <- c(1L, 2L, 3L)
typeof(x)
#> [1] "integer"
tracemem(x)
#> [1] "<0x55dfce789848>"

x[[3]] <- 4L
#> tracemem[0x55dfce789848 -> 0x55dfcc4f75d8]: eval eval withVisible withCallingHandlers eval eval with_handlers doWithOneRestart withOneRestart withRestartList doWithOneRestart withOneRestart withRestartList withRestarts <Anonymous> evaluate in_dir in_input_dir eng_r block_exec call_block process_group withCallingHandlers <Anonymous> process_file <Anonymous> <Anonymous> do.call eval eval eval eval eval.parent local
untracemem(x)

typeof(x)
#> [1] "integer"
```

To understand why this still produces a copy, here is an explanation from the [official solutions manual](https://advanced-r-solutions.rbind.io/names-and-values.html#copy-on-modify):

> Please be aware that running this code in RStudio will result in additional copies because of the reference from the environment pane.

---

**Q3.** Sketch out the relationship between the following objects:


``` r
a <- 1:10
b <- list(a, a)
c <- list(b, a, 1:10)
```

**A3.** We can understand the relationship between these objects by looking at their memory addresses:


``` r
a <- 1:10
b <- list(a, a)
c <- list(b, a, 1:10)

ref(a)
#> [1:0x55dfcd61b3f8] <int>

ref(b)
#> █ [1:0x55dfce7976b8] <list> 
#> ├─[2:0x55dfcd61b3f8] <int> 
#> └─[2:0x55dfcd61b3f8]

ref(c)
#> █ [1:0x55dfce94bc98] <list> 
#> ├─█ [2:0x55dfce7976b8] <list> 
#> │ ├─[3:0x55dfcd61b3f8] <int> 
#> │ └─[3:0x55dfcd61b3f8] 
#> ├─[3:0x55dfcd61b3f8] 
#> └─[4:0x55dfcd677a60] <int>
```

Here is what we learn:

- The name `a` references object `1:10` in the memory.
- The name `b` is bound to a list of two references to the memory address of `a`.
- The name `c` is also bound to a list of references to `a` and `b`, and `1:10` object (not bound to any name).

---

**Q4.** What happens when you run this code?


``` r
x <- list(1:10)
x[[2]] <- x
```

Draw a picture.

**A4.**


``` r
x <- list(1:10)
x
#> [[1]]
#>  [1]  1  2  3  4  5  6  7  8  9 10
obj_addr(x)
#> [1] "0x55dfcf472290"

x[[2]] <- x
x
#> [[1]]
#>  [1]  1  2  3  4  5  6  7  8  9 10
#> 
#> [[2]]
#> [[2]][[1]]
#>  [1]  1  2  3  4  5  6  7  8  9 10
obj_addr(x)
#> [1] "0x55dfcf4f1d88"

ref(x)
#> █ [1:0x55dfcf4f1d88] <list> 
#> ├─[2:0x55dfcf473fb8] <int> 
#> └─█ [3:0x55dfcf472290] <list> 
#>   └─[2:0x55dfcf473fb8]
```

I don't have access to OmniGraffle software, so I am including here the figure from the [official solution manual](https://advanced-r-solutions.rbind.io/names-and-values.html#copy-on-modify):

<img src="https://raw.githubusercontent.com/Tazinho/Advanced-R-Solutions/main/images/names_values/copy_on_modify_fig2.png" width="180pt" />

---

## Object size (Exercise 2.4.1) 

---

**Q1.** In the following example, why are `object.size(y)` and `obj_size(y)` so radically different? Consult the documentation of `object.size()`.


``` r
y <- rep(list(runif(1e4)), 100)

object.size(y)
obj_size(y)
```

**A1.** As mentioned in the docs for `object.size()`:

> This function...does not detect if elements of a list are shared.

This is why the sizes are so different:


``` r
y <- rep(list(runif(1e4)), 100)

object.size(y)
#> 8005648 bytes

obj_size(y)
#> 80.90 kB
```

---

**Q2.**  Take the following list. Why is its size somewhat misleading?


``` r
funs <- list(mean, sd, var)
obj_size(funs)
```

**A2.** These functions are not externally created objects in R, but are always available as part of base packages, so doesn't make much sense to measure their size because they are never going to be *not* available.


``` r
funs <- list(mean, sd, var)
obj_size(funs)
#> 18.76 kB
```

---

**Q3.** Predict the output of the following code:


``` r
a <- runif(1e6)
obj_size(a)

b <- list(a, a)
obj_size(b)
obj_size(a, b)

b[[1]][[1]] <- 10
obj_size(b)
obj_size(a, b)

b[[2]][[1]] <- 10
obj_size(b)
obj_size(a, b)
```

**A3.** Correctly predicted 😉


``` r
a <- runif(1e6)
obj_size(a)
#> 8.00 MB

b <- list(a, a)
obj_size(b)
#> 8.00 MB
obj_size(a, b)
#> 8.00 MB

b[[1]][[1]] <- 10
obj_size(b)
#> 16.00 MB
obj_size(a, b)
#> 16.00 MB

b[[2]][[1]] <- 10
obj_size(b)
#> 16.00 MB
obj_size(a, b)
#> 24.00 MB
```

Key pieces of information to keep in mind to make correct predictions:

- Size of empty vector


``` r
obj_size(double())
#> 48 B
```

- Size of a single double: 8 bytes


``` r
obj_size(double(1))
#> 56 B
```

- Copy-on-modify semantics

---

## Modify-in-place (Exercise 2.5.3)

---

**Q1.** Explain why the following code doesn't create a circular list.


``` r
x <- list()
x[[1]] <- x
```

**A1.** Copy-on-modify prevents the creation of a circular list.


``` r
x <- list()

obj_addr(x)
#> [1] "0x55dfcef0a640"

tracemem(x)
#> [1] "<0x55dfcef0a640>"

x[[1]] <- x
#> tracemem[0x55dfcef0a640 -> 0x55dfcefab780]: eval eval withVisible withCallingHandlers eval eval with_handlers doWithOneRestart withOneRestart withRestartList doWithOneRestart withOneRestart withRestartList withRestarts <Anonymous> evaluate in_dir in_input_dir eng_r block_exec call_block process_group withCallingHandlers <Anonymous> process_file <Anonymous> <Anonymous> do.call eval eval eval eval eval.parent local

obj_addr(x[[1]])
#> [1] "0x55dfcef0a640"
```

---

**Q2.** Wrap the two methods for subtracting medians into two functions, then use the 'bench' package to carefully compare their speeds. How does performance change as the number of columns increase?

**A2.** Let's first microbenchmark functions that do and do not create copies for varying lengths of number of columns.


``` r
library(bench)
library(tidyverse)

generateDataFrame <- function(ncol) {
  as.data.frame(matrix(runif(100 * ncol), nrow = 100))
}

withCopy <- function(ncol) {
  x <- generateDataFrame(ncol)
  medians <- vapply(x, median, numeric(1))

  for (i in seq_along(medians)) {
    x[[i]] <- x[[i]] - medians[[i]]
  }

  return(x)
}

withoutCopy <- function(ncol) {
  x <- generateDataFrame(ncol)
  medians <- vapply(x, median, numeric(1))

  y <- as.list(x)

  for (i in seq_along(medians)) {
    y[[i]] <- y[[i]] - medians[[i]]
  }

  return(y)
}

benchComparison <- function(ncol) {
  bench::mark(
    withCopy(ncol),
    withoutCopy(ncol),
    iterations = 100,
    check = FALSE
  ) %>%
    dplyr::select(expression:total_time)
}

nColList <- list(1, 10, 50, 100, 250, 500, 1000)

names(nColList) <- as.character(nColList)

benchDf <- purrr::map_dfr(
  .x = nColList,
  .f = benchComparison,
  .id = "nColumns"
)
```

Plotting these benchmarks reveals how the performance gets increasingly worse as the number of data frames increases:


``` r
ggplot(
  benchDf,
  aes(
    x = as.numeric(nColumns),
    y = median,
    group = as.character(expression),
    color = as.character(expression)
  )
) +
  geom_line() +
  labs(
    x = "Number of Columns",
    y = "Median Execution Time (ms)",
    colour = "Type of function"
  )
```

<img src="Names-values_files/figure-epub3/Names-values-31-1.png" width="100%" />

---

**Q3.** What happens if you attempt to use `tracemem()` on an environment?

**A3.** It doesn't work and the documentation for `tracemem()` makes it clear why:

> It is not useful to trace `NULL`, environments, promises, weak references, or external pointer objects, as these are not duplicated


``` r
e <- rlang::env(a = 1, b = "3")
tracemem(e)
#> Error in tracemem(e): 'tracemem' is not useful for promise and environment objects
```

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
#>  date     2025-04-27
#>  pandoc   3.6.4 @ /opt/hostedtoolcache/pandoc/3.6.4/x64/ (via rmarkdown)
#>  quarto   NA
#> 
#> ─ Packages ───────────────────────────────────────────────
#>  package      * version date (UTC) lib source
#>  base         * 4.5.0   2025-04-26 [3] local
#>  bench        * 1.1.4   2025-01-16 [1] RSPM
#>  bookdown       0.43    2025-04-15 [1] RSPM
#>  cli            3.6.5   2025-04-23 [1] RSPM
#>  compiler       4.5.0   2025-04-26 [3] local
#>  crayon         1.5.3   2024-06-20 [1] RSPM
#>  datasets     * 4.5.0   2025-04-26 [3] local
#>  digest         0.6.37  2024-08-19 [1] RSPM
#>  dplyr        * 1.1.4   2023-11-17 [1] RSPM
#>  emoji          16.0.0  2024-10-28 [1] RSPM
#>  evaluate       1.0.3   2025-01-10 [1] RSPM
#>  farver         2.1.2   2024-05-13 [1] RSPM
#>  fastmap        1.2.0   2024-05-15 [1] RSPM
#>  forcats      * 1.0.0   2023-01-29 [1] RSPM
#>  generics       0.1.3   2022-07-05 [1] RSPM
#>  ggplot2      * 3.5.2   2025-04-09 [1] RSPM
#>  glue           1.8.0   2024-09-30 [1] RSPM
#>  graphics     * 4.5.0   2025-04-26 [3] local
#>  grDevices    * 4.5.0   2025-04-26 [3] local
#>  grid           4.5.0   2025-04-26 [3] local
#>  gtable         0.3.6   2024-10-25 [1] RSPM
#>  hms            1.1.3   2023-03-21 [1] RSPM
#>  htmltools      0.5.8.1 2024-04-04 [1] RSPM
#>  knitr          1.50    2025-03-16 [1] RSPM
#>  labeling       0.4.3   2023-08-29 [1] RSPM
#>  lifecycle      1.0.4   2023-11-07 [1] RSPM
#>  lobstr       * 1.1.2   2022-06-22 [1] RSPM
#>  lubridate    * 1.9.4   2024-12-08 [1] RSPM
#>  magrittr     * 2.0.3   2022-03-30 [1] RSPM
#>  methods      * 4.5.0   2025-04-26 [3] local
#>  pillar         1.10.2  2025-04-05 [1] RSPM
#>  pkgconfig      2.0.3   2019-09-22 [1] RSPM
#>  prettyunits    1.2.0   2023-09-24 [1] RSPM
#>  profmem        0.6.0   2020-12-13 [1] RSPM
#>  purrr        * 1.0.4   2025-02-05 [1] RSPM
#>  R6             2.6.1   2025-02-15 [1] RSPM
#>  RColorBrewer   1.1-3   2022-04-03 [1] RSPM
#>  readr        * 2.1.5   2024-01-10 [1] RSPM
#>  rlang          1.1.6   2025-04-11 [1] RSPM
#>  rmarkdown      2.29    2024-11-04 [1] RSPM
#>  scales         1.4.0   2025-04-24 [1] RSPM
#>  sessioninfo    1.2.3   2025-02-05 [1] RSPM
#>  stats        * 4.5.0   2025-04-26 [3] local
#>  stringi        1.8.7   2025-03-27 [1] RSPM
#>  stringr      * 1.5.1   2023-11-14 [1] RSPM
#>  tibble       * 3.2.1   2023-03-20 [1] RSPM
#>  tidyr        * 1.3.1   2024-01-24 [1] RSPM
#>  tidyselect     1.2.1   2024-03-11 [1] RSPM
#>  tidyverse    * 2.0.0   2023-02-22 [1] RSPM
#>  timechange     0.3.0   2024-01-18 [1] RSPM
#>  tools          4.5.0   2025-04-26 [3] local
#>  tzdb           0.5.0   2025-03-15 [1] RSPM
#>  utils        * 4.5.0   2025-04-26 [3] local
#>  vctrs          0.6.5   2023-12-01 [1] RSPM
#>  withr          3.0.2   2024-10-28 [1] RSPM
#>  xfun           0.52    2025-04-02 [1] RSPM
#>  yaml           2.3.10  2024-07-26 [1] RSPM
#> 
#>  [1] /home/runner/work/_temp/Library
#>  [2] /opt/R/4.5.0/lib/R/site-library
#>  [3] /opt/R/4.5.0/lib/R/library
#>  * ── Packages attached to the search path.
#> 
#> ──────────────────────────────────────────────────────────
```
