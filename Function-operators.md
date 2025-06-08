# Function operators



Attaching the needed libraries:


``` r
library(purrr, warn.conflicts = FALSE)
```

## Existing function operators (Exercises 11.2.3)

---

**Q1.** Base R provides a function operator in the form of `Vectorize()`. What does it do? When might you use it?

**A1.** `Vectorize()` function creates a function that vectorizes the action of the provided function over specified arguments (i.e., it acts on each element of the vector). We will see its utility by solving a problem that otherwise would be difficult to solve.

The problem is to find indices of matching numeric values for the given threshold by creating a hybrid of the following functions:

- `%in%` (which doesn't provide any way to provide tolerance when comparing numeric values),
- `dplyr::near()` (which is vectorized element-wise and thus expects two vectors of equal length)


``` r
which_near <- function(x, y, tolerance) {
  # Vectorize `dplyr::near()` function only over the `y` argument.
  # `Vectorize()` is a function operator and will return a function.
  customNear <- Vectorize(dplyr::near, vectorize.args = c("y"), SIMPLIFY = FALSE)

  # Apply the vectorized function to vector arguments and then check where the
  # comparisons are equal (i.e. `TRUE`) using `which()`.
  #
  # Use `compact()` to remove empty elements from the resulting list.
  index_list <- purrr::compact(purrr::map(customNear(x, y, tol = tolerance), which))

  # If there are any matches, return the indices as an atomic vector of integers.
  if (length(index_list) > 0L) {
    index_vector <- purrr::simplify(index_list, "integer")
    return(index_vector)
  }

  # If there are no matches
  return(integer(0L))
}
```
 
Let's use it:


``` r
x1 <- c(2.1, 3.3, 8.45, 8, 6)
x2 <- c(6, 8.40, 3)

which_near(x1, x2, tolerance = 0.1)
#> [1] 5 3
```

Note that we needed to create a new function for this because neither of the existing functions do what we want.


``` r
which(x1 %in% x2)
#> [1] 5

which(dplyr::near(x1, x2, tol = 0.1))
#> Warning in x - y: longer object length is not a multiple of
#> shorter object length
#> integer(0)
```

We solved a complex task here using the `Vectorize()` function!

---

**Q2.** Read the source code for `possibly()`. How does it work?

**A2.** Let's have a look at the source code for this function:


``` r
possibly
#> function (.f, otherwise = NULL, quiet = TRUE) 
#> {
#>     .f <- as_mapper(.f)
#>     force(otherwise)
#>     check_bool(quiet)
#>     function(...) {
#>         tryCatch(.f(...), error = function(e) {
#>             if (!quiet) 
#>                 message("Error: ", conditionMessage(e))
#>             otherwise
#>         })
#>     }
#> }
#> <bytecode: 0x55ff86a665d8>
#> <environment: namespace:purrr>
```

Looking at this code, we can see that `possibly()`:

- uses `tryCatch()` for error handling
- has a parameter `otherwise` to specify default value in case an error occurs
- has a parameter `quiet` to suppress error message (if needed)

---

**Q3.** Read the source code for `safely()`. How does it work?

**A3.** Let's have a look at the source code for this function:


``` r
safely
#> function (.f, otherwise = NULL, quiet = TRUE) 
#> {
#>     .f <- as_mapper(.f)
#>     force(otherwise)
#>     check_bool(quiet)
#>     function(...) capture_error(.f(...), otherwise, quiet)
#> }
#> <bytecode: 0x55ff86c50358>
#> <environment: namespace:purrr>

purrr:::capture_error
#> function (code, otherwise = NULL, quiet = TRUE) 
#> {
#>     tryCatch(list(result = code, error = NULL), error = function(e) {
#>         if (!quiet) 
#>             message("Error: ", conditionMessage(e))
#>         list(result = otherwise, error = e)
#>     })
#> }
#> <bytecode: 0x55ff86c883f8>
#> <environment: namespace:purrr>
```

Looking at this code, we can see that `safely()`:

- uses a list to save both the results (if the function executes successfully) and the error (if it fails)
- uses `tryCatch()` for error handling
- has a parameter `otherwise` to specify default value in case an error occurs
- has a parameter `quiet` to suppress error message (if needed)

---

## Case study: Creating your own function operators (Exercises 11.3.1)

---

**Q1.** Weigh the pros and cons of `download.file %>% dot_every(10) %>% delay_by(0.1)` versus `download.file %>% delay_by(0.1) %>% dot_every(10)`.

**A1.** Although both of these chains of piped operations produce the same number of dots and would need the same amount of time, there is a subtle difference in how they do this.

- `download.file %>% dot_every(10) %>% delay_by(0.1)`

Here, the printing of the dot is also delayed, and the first dot is printed when the 10th URL download starts.

- `download.file %>% delay_by(0.1) %>% dot_every(10)`

Here, the first dot is printed after the 9th download is finished, and the 10th download starts after a short delay.

---

**Q2.** Should you memoise `download.file()`? Why or why not?

**A2.** Since `download.file()` is meant to download files from the Internet, memoising it is not recommended for the following reasons:

- Memoization is helpful when giving the same input the function returns the same output. This is not necessarily the case for webpages since they constantly change, and you may continue to "download" an outdated version of the webpage.

- Memoization works by caching results, which can take up a significant amount of memory. 

---

**Q3.** Create a function operator that reports whenever a file is created or deleted in the working directory, using `dir()` and `setdiff()`. What other global function effects might you want to track?

**A3.** First, let's create helper functions to compare and print added or removed filenames:


``` r
print_multiple_entries <- function(header, entries) {
  message(paste0(header, ":\n"), paste0(entries, collapse = "\n"))
}

file_comparator <- function(old, new) {
  if (setequal(old, new)) {
    return()
  }

  removed <- setdiff(old, new)
  added <- setdiff(new, old)

  if (length(removed) > 0L) print_multiple_entries("- File removed", removed)
  if (length(added) > 0L) print_multiple_entries("- File added", added)
}
```

We can then write a function operator and use it to create functions that will do the necessary tracking:


``` r
dir_tracker <- function(f) {
  force(f)
  function(...) {
    old_files <- dir()
    on.exit(file_comparator(old_files, dir()), add = TRUE)

    f(...)
  }
}

file_creation_tracker <- dir_tracker(file.create)
file_deletion_tracker <- dir_tracker(file.remove)
```

Let's try it out:


``` r
file_creation_tracker(c("a.txt", "b.txt"))
#> - File added:
#> a.txt
#> b.txt
#> [1] TRUE TRUE

file_deletion_tracker(c("a.txt", "b.txt"))
#> - File removed:
#> a.txt
#> b.txt
#> [1] TRUE TRUE
```

Other global function effects we might want to track:

- working directory
- environment variables
- connections
- library paths
- graphics devices
- [etc.](https://withr.r-lib.org/reference/index.html)

---

**Q4.** Write a function operator that logs a timestamp and message to a file every time a function is run.

**A4.** The following function operator logs a timestamp and message to a file every time a function is run:


``` r
# helper function to write to a file connection
write_line <- function(filepath, ...) {
  cat(..., "\n", sep = "", file = filepath, append = TRUE)
}

# function operator
logger <- function(f, filepath) {
  force(f)
  force(filepath)

  write_line(filepath, "Function created at: ", as.character(Sys.time()))

  function(...) {
    write_line(filepath, "Function called at:  ", as.character(Sys.time()))
    f(...)
  }
}

# check that the function works as expected with a tempfile
withr::with_tempfile("logfile", code = {
  logged_runif <- logger(runif, logfile)

  Sys.sleep(sample.int(10, 1))
  logged_runif(1)

  Sys.sleep(sample.int(10, 1))
  logged_runif(2)

  Sys.sleep(sample.int(10, 1))
  logged_runif(3)

  cat(readLines(logfile), sep = "\n")
})
#> Function created at: 2025-06-08 00:20:40.012582
#> Function called at:  2025-06-08 00:20:45.019423
#> Function called at:  2025-06-08 00:20:50.024789
#> Function called at:  2025-06-08 00:20:58.029587
```

---

**Q5.** Modify `delay_by()` so that instead of delaying by a fixed amount of time, it ensures that a certain amount of time has elapsed since the function was last called. That is, if you called `g <- delay_by(1, f); g(); Sys.sleep(2); g()` there shouldn't be an extra delay.

**A5.** Modified version of the function meeting the specified requirements:


``` r
delay_by_atleast <- function(f, amount) {
  force(f)
  force(amount)

  # the last time the function was run
  last_time <- NULL

  function(...) {
    if (!is.null(last_time)) {
      wait <- (last_time - Sys.time()) + amount
      if (wait > 0) Sys.sleep(wait)
    }

    # update the time in the parent frame for the next run when the function finishes
    on.exit(last_time <<- Sys.time())

    f(...)
  }
}
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
#>  date     2025-06-08
#>  pandoc   3.7.0.2 @ /opt/hostedtoolcache/pandoc/3.7.0.2/x64/ (via rmarkdown)
#>  quarto   NA
#> 
#> ─ Packages ───────────────────────────────────────────────
#>  package     * version date (UTC) lib source
#>  base        * 4.5.0   2025-04-28 [3] local
#>  bookdown      0.43    2025-04-15 [1] RSPM
#>  cli           3.6.5   2025-04-23 [1] RSPM
#>  compiler      4.5.0   2025-04-28 [3] local
#>  datasets    * 4.5.0   2025-04-28 [3] local
#>  digest        0.6.37  2024-08-19 [1] RSPM
#>  dplyr         1.1.4   2023-11-17 [1] RSPM
#>  emoji         16.0.0  2024-10-28 [1] RSPM
#>  evaluate      1.0.3   2025-01-10 [1] RSPM
#>  fastmap       1.2.0   2024-05-15 [1] RSPM
#>  generics      0.1.4   2025-05-09 [1] RSPM
#>  glue          1.8.0   2024-09-30 [1] RSPM
#>  graphics    * 4.5.0   2025-04-28 [3] local
#>  grDevices   * 4.5.0   2025-04-28 [3] local
#>  htmltools     0.5.8.1 2024-04-04 [1] RSPM
#>  knitr         1.50    2025-03-16 [1] RSPM
#>  lifecycle     1.0.4   2023-11-07 [1] RSPM
#>  magrittr    * 2.0.3   2022-03-30 [1] RSPM
#>  methods     * 4.5.0   2025-04-28 [3] local
#>  pillar        1.10.2  2025-04-05 [1] RSPM
#>  pkgconfig     2.0.3   2019-09-22 [1] RSPM
#>  purrr       * 1.0.4   2025-02-05 [1] RSPM
#>  R6            2.6.1   2025-02-15 [1] RSPM
#>  rlang         1.1.6   2025-04-11 [1] RSPM
#>  rmarkdown     2.29    2024-11-04 [1] RSPM
#>  sessioninfo   1.2.3   2025-02-05 [1] RSPM
#>  stats       * 4.5.0   2025-04-28 [3] local
#>  stringi       1.8.7   2025-03-27 [1] RSPM
#>  stringr       1.5.1   2023-11-14 [1] RSPM
#>  tibble        3.2.1   2023-03-20 [1] RSPM
#>  tidyselect    1.2.1   2024-03-11 [1] RSPM
#>  tools         4.5.0   2025-04-28 [3] local
#>  utils       * 4.5.0   2025-04-28 [3] local
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
