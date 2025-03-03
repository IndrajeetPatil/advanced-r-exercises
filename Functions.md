# Functions



Attaching the needed libraries:


``` r
library(tidyverse, warn.conflicts = FALSE)
```

## Function fundamentals (Exercises 6.2.5)

**Q1.** Given a name, like `"mean"`, `match.fun()` lets you find a function. Given a function, can you find its name? Why doesn't that make sense in R?

**A1.** Given a name, `match.fun()` lets you find a function.


``` r
match.fun("mean")
#> function (x, ...) 
#> UseMethod("mean")
#> <bytecode: 0x564505d220c0>
#> <environment: namespace:base>
```

But, given a function, it doesn't make sense to find its name because there can be multiple names bound to the same function.


``` r
f1 <- function(x) mean(x)
f2 <- f1

match.fun("f1")
#> function (x) 
#> mean(x)

match.fun("f2")
#> function (x) 
#> mean(x)
```

**Q2.** It's possible (although typically not useful) to call an anonymous function. Which of the two approaches below is correct? Why?


``` r
function(x) 3()
#> function (x) 
#> 3()
(function(x) 3)()
#> [1] 3
```

**A2.** The first expression is not correct since the function will evaluate `3()`, which is syntactically not allowed since literals can't be treated like functions.


``` r
f <- (function(x) 3())
f
#> function (x) 
#> 3()
f()
#> Error in f(): attempt to apply non-function

rlang::is_syntactic_literal(3)
#> [1] TRUE
```

This is the correct way to call an anonymous function.


``` r
g <- (function(x) 3)
g
#> function (x) 
#> 3
g()
#> [1] 3
```

**Q3.** A good rule of thumb is that an anonymous function should fit on one line and shouldn't need to use `{}`. Review your code. Where could you have used an anonymous function instead of a named function? Where should you have used a named function instead of an anonymous function?

**A3.** Self activity.

**Q4.** What function allows you to tell if an object is a function? What function allows you to tell if a function is a primitive function?

**A4.** Use `is.function()` to check if an *object* is a *function*:


``` r
# these are functions
f <- function(x) 3
is.function(mean)
#> [1] TRUE
is.function(f)
#> [1] TRUE

# these aren't
is.function("x")
#> [1] FALSE
is.function(new.env())
#> [1] FALSE
```

Use `is.primitive()` to check if a *function* is *primitive*:


``` r
# primitive
is.primitive(sum)
#> [1] TRUE
is.primitive(`+`)
#> [1] TRUE

# not primitive
is.primitive(mean)
#> [1] FALSE
is.primitive(read.csv)
#> [1] FALSE
```

**Q5.** This code makes a list of all functions in the base package. 


``` r
objs <- mget(ls("package:base", all = TRUE), inherits = TRUE)
funs <- Filter(is.function, objs)
```

Use it to answer the following questions:

a. Which base function has the most arguments?

b. How many base functions have no arguments? What's special about those functions?

c. How could you adapt the code to find all primitive functions?

**A5.** The provided code is the following:


``` r
objs <- mget(ls("package:base", all = TRUE), inherits = TRUE)
funs <- Filter(is.function, objs)
```

a. Which base function has the most arguments?

We can use `formals()` to extract number of arguments, but because this function returns `NULL` for primitive functions.


``` r
formals("!")
#> NULL

length(formals("!"))
#> [1] 0
```

Therefore, we will focus only on non-primitive functions.


``` r
funs <- purrr::discard(funs, is.primitive)
```

`scan()` function has the most arguments.


``` r
df_formals <- purrr::map_df(funs, ~ length(formals(.))) %>%
  tidyr::pivot_longer(
    cols = dplyr::everything(),
    names_to = "function",
    values_to = "argumentCount"
  ) %>%
  dplyr::arrange(desc(argumentCount))

df_formals
#> # A tibble: 1,145 × 2
#>    `function`        argumentCount
#>    <chr>                     <int>
#>  1 scan                         22
#>  2 source                       17
#>  3 format.default               16
#>  4 formatC                      15
#>  5 library                      13
#>  6 merge.data.frame             13
#>  7 prettyNum                    13
#>  8 system2                      12
#>  9 system                       11
#> 10 all.equal.numeric            10
#> # ℹ 1,135 more rows
```

b. How many base functions have no arguments? What’s special about those functions?



At the time of writing, 47 base (non-primitive) functions have no arguments. 


``` r
dplyr::filter(df_formals, argumentCount == 0)
#> # A tibble: 47 × 2
#>    `function`               argumentCount
#>    <chr>                            <int>
#>  1 .First.sys                           0
#>  2 .NotYetImplemented                   0
#>  3 .OptRequireMethods                   0
#>  4 .standard_regexps                    0
#>  5 .tryResumeInterrupt                  0
#>  6 closeAllConnections                  0
#>  7 contributors                         0
#>  8 Cstack_info                          0
#>  9 default.stringsAsFactors             0
#> 10 extSoftVersion                       0
#> # ℹ 37 more rows
```

c. How could you adapt the code to find all primitive functions?


``` r
objs <- mget(ls("package:base", all = TRUE), inherits = TRUE)
funs <- Filter(is.function, objs)
primitives <- Filter(is.primitive, funs)

length(primitives)
#> [1] 210

names(primitives)
#>   [1] "-"                    ":"                   
#>   [3] "::"                   ":::"                 
#>   [5] "!"                    "!="                  
#>   [7] "...elt"               "...length"           
#>   [9] "...names"             ".C"                  
#>  [11] ".cache_class"         ".Call"               
#>  [13] ".Call.graphics"       ".class2"             
#>  [15] ".External"            ".External.graphics"  
#>  [17] ".External2"           ".Fortran"            
#>  [19] ".Internal"            ".isMethodsDispatchOn"
#>  [21] ".Primitive"           ".primTrace"          
#>  [23] ".primUntrace"         ".subset"             
#>  [25] ".subset2"             "("                   
#>  [27] "["                    "[["                  
#>  [29] "[[<-"                 "[<-"                 
#>  [31] "{"                    "@"                   
#>  [33] "@<-"                  "*"                   
#>  [35] "/"                    "&"                   
#>  [37] "&&"                   "%*%"                 
#>  [39] "%/%"                  "%%"                  
#>  [41] "^"                    "+"                   
#>  [43] "<"                    "<-"                  
#>  [45] "<<-"                  "<="                  
#>  [47] "="                    "=="                  
#>  [49] ">"                    ">="                  
#>  [51] "|"                    "||"                  
#>  [53] "~"                    "$"                   
#>  [55] "$<-"                  "abs"                 
#>  [57] "acos"                 "acosh"               
#>  [59] "all"                  "any"                 
#>  [61] "anyNA"                "Arg"                 
#>  [63] "as.call"              "as.character"        
#>  [65] "as.complex"           "as.double"           
#>  [67] "as.environment"       "as.integer"          
#>  [69] "as.logical"           "as.numeric"          
#>  [71] "as.raw"               "asin"                
#>  [73] "asinh"                "atan"                
#>  [75] "atanh"                "attr"                
#>  [77] "attr<-"               "attributes"          
#>  [79] "attributes<-"         "baseenv"             
#>  [81] "break"                "browser"             
#>  [83] "c"                    "call"                
#>  [85] "ceiling"              "class"               
#>  [87] "class<-"              "Conj"                
#>  [89] "cos"                  "cosh"                
#>  [91] "cospi"                "crossprod"           
#>  [93] "cummax"               "cummin"              
#>  [95] "cumprod"              "cumsum"              
#>  [97] "declare"              "digamma"             
#>  [99] "dim"                  "dim<-"               
#> [101] "dimnames"             "dimnames<-"          
#> [103] "emptyenv"             "enc2native"          
#> [105] "enc2utf8"             "environment<-"       
#> [107] "Exec"                 "exp"                 
#> [109] "expm1"                "expression"          
#> [111] "floor"                "for"                 
#> [113] "forceAndCall"         "function"            
#> [115] "gamma"                "gc.time"             
#> [117] "globalenv"            "if"                  
#> [119] "Im"                   "interactive"         
#> [121] "invisible"            "is.array"            
#> [123] "is.atomic"            "is.call"             
#> [125] "is.character"         "is.complex"          
#> [127] "is.double"            "is.environment"      
#> [129] "is.expression"        "is.finite"           
#> [131] "is.function"          "is.infinite"         
#> [133] "is.integer"           "is.language"         
#> [135] "is.list"              "is.logical"          
#> [137] "is.matrix"            "is.na"               
#> [139] "is.name"              "is.nan"              
#> [141] "is.null"              "is.numeric"          
#> [143] "is.object"            "is.pairlist"         
#> [145] "is.raw"               "is.recursive"        
#> [147] "is.single"            "is.symbol"           
#> [149] "isS4"                 "lazyLoadDBfetch"     
#> [151] "length"               "length<-"            
#> [153] "levels<-"             "lgamma"              
#> [155] "list"                 "log"                 
#> [157] "log10"                "log1p"               
#> [159] "log2"                 "max"                 
#> [161] "min"                  "missing"             
#> [163] "Mod"                  "names"               
#> [165] "names<-"              "nargs"               
#> [167] "next"                 "nzchar"              
#> [169] "oldClass"             "oldClass<-"          
#> [171] "on.exit"              "pos.to.env"          
#> [173] "proc.time"            "prod"                
#> [175] "quote"                "range"               
#> [177] "Re"                   "rep"                 
#> [179] "repeat"               "retracemem"          
#> [181] "return"               "round"               
#> [183] "seq_along"            "seq_len"             
#> [185] "seq.int"              "sign"                
#> [187] "signif"               "sin"                 
#> [189] "sinh"                 "sinpi"               
#> [191] "sqrt"                 "standardGeneric"     
#> [193] "storage.mode<-"       "substitute"          
#> [195] "sum"                  "switch"              
#> [197] "Tailcall"             "tan"                 
#> [199] "tanh"                 "tanpi"               
#> [201] "tcrossprod"           "tracemem"            
#> [203] "trigamma"             "trunc"               
#> [205] "unCfillPOSIXlt"       "unclass"             
#> [207] "untracemem"           "UseMethod"           
#> [209] "while"                "xtfrm"
```

**Q6.** What are the three important components of a function?

**A6.** Except for primitive functions, all functions have 3 important components:

* `formals()`
* `body()`
* `environment()`

**Q7.** When does printing a function not show the environment it was created in?

**A7.** All package functions print their environment:


``` r
# base
mean
#> function (x, ...) 
#> UseMethod("mean")
#> <bytecode: 0x564505d220c0>
#> <environment: namespace:base>

# other package function
purrr::map
#> function (.x, .f, ..., .progress = FALSE) 
#> {
#>     map_("list", .x, .f, ..., .progress = .progress)
#> }
#> <bytecode: 0x56450b5aa718>
#> <environment: namespace:purrr>
```

There are two exceptions where the enclosing environment won't be printed:

- primitive functions


``` r
sum
#> function (..., na.rm = FALSE)  .Primitive("sum")
```

- functions created in the global environment


``` r
f <- function(x) mean(x)
f
#> function (x) 
#> mean(x)
```

## Lexical scoping (Exercises 6.4.5)

**Q1.** What does the following code return? Why? Describe how each of the three `c`'s is interpreted.


``` r
c <- 10
c(c = c)
```

**A1.** In `c(c = c)`:

* first *c* is interpreted as a function call `c()`
* second *c* as a name for the vector element
* third *c* as a variable with value `10`


``` r
c <- 10
c(c = c)
#>  c 
#> 10
```

You can also see this in the lexical analysis of this expression:


``` r
p_expr <- parse(text = "c(c = c)", keep.source = TRUE)
getParseData(p_expr) %>% select(token, text)
#>                   token text
#> 12                 expr     
#> 1  SYMBOL_FUNCTION_CALL    c
#> 3                  expr     
#> 2                   '('    (
#> 4            SYMBOL_SUB    c
#> 5                EQ_SUB    =
#> 6                SYMBOL    c
#> 8                  expr     
#> 7                   ')'    )
```

**Q2.** What are the four principles that govern how R looks for values?

**A2.** Principles that govern how R looks for values:

1. Name masking (names defined inside a function mask names defined outside a function)

1. Functions vs. variables (the rule above also applies to function names) 

1. A fresh start (every time a function is called, a new environment is created to host its execution)

1. Dynamic look-up (R looks for values when the function is run, not when the function is created)

**Q3.** What does the following function return? Make a prediction before running the code yourself.


``` r
f <- function(x) {
  f <- function(x) {
    f <- function() {
      x^2
    }
    f() + 1
  }
  f(x) * 2
}
f(10)
```

**A3.** Correctly predicted 😉


``` r
f <- function(x) {
  f <- function(x) {
    f <- function() {
      x^2
    }
    f() + 1
  }
  f(x) * 2
}

f(10)
#> [1] 202
```

Although there are multiple `f()` functions, the order of evaluation goes from inside to outside with `x^2` evaluated first and `f(x) * 2` evaluated last. This results in 202 (= `((10 ^ 2) + 1) * 2`).

## Lazy evaluation (Exercises 6.5.4)

**Q1.** What important property of `&&` makes `x_ok()` work?


``` r
x_ok <- function(x) {
  !is.null(x) && length(x) == 1 && x > 0
}

x_ok(NULL)
x_ok(1)
x_ok(1:3)
```

What is different with this code? Why is this behaviour undesirable here?
    

``` r
x_ok <- function(x) {
  !is.null(x) & length(x) == 1 & x > 0
}

x_ok(NULL)
x_ok(1)
x_ok(1:3)
```

**A1.** `&&` evaluates left to right and has short-circuit evaluation, i.e., if the first operand is `TRUE`, R will short-circuit and not even look at the second operand.


``` r
x_ok <- function(x) {
  !is.null(x) && length(x) == 1 && x > 0
}

x_ok(NULL)
#> [1] FALSE

x_ok(1)
#> [1] TRUE

x_ok(1:3)
#> [1] FALSE
```

Replacing `&&` with `&` is undesirable because it performs element-wise logical comparisons and returns a vector of values that is not always useful for a decision (`TRUE`, `FALSE`, or `NA`).


``` r
x_ok <- function(x) {
  !is.null(x) & length(x) == 1 & x > 0
}

x_ok(NULL)
#> logical(0)

x_ok(1)
#> [1] TRUE

x_ok(1:3)
#> [1] FALSE FALSE FALSE
```

**Q2.** What does this function return? Why? Which principle does it illustrate?


``` r
f2 <- function(x = z) {
  z <- 100
  x
}
f2()
```

**A2.** The function returns `100` due to lazy evaluation. 

When function execution environment encounters `x`, it evaluates argument `x = z` and since the name `z` is already bound to the value 100 in this environment, `x` is also bound to the same value.

We can check this by looking at the memory addresses:


``` r
f2 <- function(x = z) {
  z <- 100
  print(lobstr::obj_addrs(list(x, z)))
  x
}

f2()
#> [1] "0x56450bc730a0" "0x56450bc730a0"
#> [1] 100
```

**Q3.** What does this function return? Why? Which principle does it illustrate?
  

``` r
y <- 10
f1 <- function(x =
                 {
                   y <- 1
                   2
                 },
               y = 0) {
  c(x, y)
}
f1()
y
```

**A3.** Let's first look at what the function returns:


``` r
y <- 10
f1 <- function(x =
                 {
                   y <- 1
                   2
                 },
               y = 0) {
  c(x, y)
}
f1()
#> [1] 2 1
y
#> [1] 10
```

This is because of name masking. In the function call `c(x, y)`, when `x` is accessed in the function environment, the following promise is evaluated in the function environment:


``` r
x <- {
  y <- 1
  2
}
```

And, thus `y` gets assigned to `1`, and `x` to `2`, since its the last value in that scope.

Therefore, neither the promise `y = 0` nor global assignment `y <- 10` is ever consulted to find the value for `y`.

**Q4.** In `hist()`, the default value of `xlim` is `range(breaks)`, the default value for `breaks` is `"Sturges"`, and


``` r
range("Sturges")
#> [1] "Sturges" "Sturges"
```

Explain how `hist()` works to get a correct `xlim` value.

**A4.** The `xlim` defines the range of the histogram's `x`-axis.


``` r
hist(mtcars$wt, xlim = c(1, 6))
```

<img src="Functions_files/figure-epub3/Functions-36-1.png" width="100%" />

The default `xlim = range(breaks)` and `breaks = "Sturges"` arguments reveal that the function uses Sturges' algorithm to compute the number of breaks.


``` r
nclass.Sturges(mtcars$wt)
#> [1] 6
```

To see the implementation, run `sloop::s3_get_method("hist.default")`.

`hist()` ensures that the chosen algorithm returns a numeric vector containing at least two unique elements before `xlim` is computed.

**Q5.** Explain why this function works. Why is it confusing?


``` r
show_time <- function(x = stop("Error!")) {
  stop <- function(...) Sys.time()
  print(x)
}

show_time()
#> [1] "2025-03-03 18:28:39 UTC"
```

**A5.** Let's take this step-by-step.

The function argument `x` is missing in the function call. This means that `stop("Error!")` is evaluated in the function environment, and not global environment.

But, due to lazy evaluation, the promise `stop("Error!")` is evaluated only when `x` is accessed. This happens only when `print(x)` is called.

`print(x)` leads to `x` being evaluated, which evaluates `stop` in the function environment. But, in function environment, the `base::stop()` is masked by a locally defined `stop()` function, which returns `Sys.time()` output.

**Q6.** How many arguments are required when calling `library()`?

**A6.** Going solely by its signature, 


``` r
formals(library)
#> $package
#> 
#> 
#> $help
#> 
#> 
#> $pos
#> [1] 2
#> 
#> $lib.loc
#> NULL
#> 
#> $character.only
#> [1] FALSE
#> 
#> $logical.return
#> [1] FALSE
#> 
#> $warn.conflicts
#> 
#> 
#> $quietly
#> [1] FALSE
#> 
#> $verbose
#> getOption("verbose")
#> 
#> $mask.ok
#> 
#> 
#> $exclude
#> 
#> 
#> $include.only
#> 
#> 
#> $attach.required
#> missing(include.only)
```

it looks like the following arguments are required:


``` r
formals(library) %>%
  purrr::discard(is.null) %>%
  purrr::map_lgl(~ .x == "") %>%
  purrr::keep(~ isTRUE(.x)) %>%
  names()
#> [1] "package"        "help"           "warn.conflicts"
#> [4] "mask.ok"        "exclude"        "include.only"
```

But, in reality, only one argument is required: `package`. The function internally checks if the other arguments are missing and adjusts accordingly.

It would have been better if there arguments were `NULL` instead of missing; that would avoid this confusion.

## `...` (dot-dot-dot) (Exercises 6.6.1)

**Q1.** Explain the following results:


``` r
sum(1, 2, 3)
#> [1] 6
mean(1, 2, 3)
#> [1] 1

sum(1, 2, 3, na.omit = TRUE)
#> [1] 7
mean(1, 2, 3, na.omit = TRUE)
#> [1] 1
```

**A1.** Let's look at arguments for these functions:


``` r
str(sum)
#> function (..., na.rm = FALSE)
str(mean)
#> function (x, ...)
```

As can be seen, `sum()` function doesn't have `na.omit` argument. So, the input `na.omit = TRUE` is treated as `1` (logical implicitly coerced to numeric), and thus the results. So, the expression evaluates to `sum(1, 2, 3, 1)`.

For `mean()` function, there is only one parameter (`x`) and it's matched by the first argument (`1`). So, the expression evaluates to `mean(1)`.

**Q2.** Explain how to find the documentation for the named arguments in the following function call:


``` r
plot(1:10, col = "red", pch = 20, xlab = "x", col.lab = "blue")
```

<img src="Functions_files/figure-epub3/Functions-43-1.png" width="100%" />

**A2.** Typing `?plot` in the console, we see its documentation, which also shows its signature:


```
#> function (x, y, ...)
```

Since `...` are passed to `par()`, we can look at `?par` docs:


```
#> function (..., no.readonly = FALSE)
```

And so on.

The docs for all parameters of interest [reside there](https://rdrr.io/r/graphics/par.html).

**Q3.** Why does `plot(1:10, col = "red")` only colour the points, not the axes or labels? Read the source code of `plot.default()` to find out.

**A3.** Source code can be found [here](https://github.com/wch/r-source/blob/79e73dba5259b25ec30118d45fea64aeac0f41dc/src/library/graphics/R/plot.R#L51-L84).

`plot.default()` passes `...` to `localTitle()`, which passes it to `title()`.

`title()` has four parts: `main`, `sub`, `xlab`, `ylab`.

So having a single argument `col` would not work as it will be ambiguous as to which element to apply this argument to.


``` r
localTitle <- function(..., col, bg, pch, cex, lty, lwd) title(...)

title <- function(main = NULL,
  sub = NULL,
  xlab = NULL,
  ylab = NULL,
  line = NA,
  outer = FALSE,
  ...) {
  main <- as.graphicsAnnot(main)
  sub <- as.graphicsAnnot(sub)
  xlab <- as.graphicsAnnot(xlab)
  ylab <- as.graphicsAnnot(ylab)
  .External.graphics(C_title, main, sub, xlab, ylab, line, outer, ...)
  invisible()
}
```

## Exiting a function (Exercises 6.7.5)

**Q1.** What does `load()` return? Why don't you normally see these values?

**A1.** The `load()` function reloads datasets that were saved using the `save()` function:


``` r
save(iris, file = "my_iris.rda")
load("my_iris.rda")
```

We normally don't see any value because the function loads the datasets invisibly.

We can change this by setting `verbose = TRUE`:


``` r
load("my_iris.rda", verbose = TRUE)
#> Loading objects:
#>   iris

# cleanup
unlink("my_iris.rda")
```

**Q2.** What does `write.table()` return? What would be more useful?

**A2.** The `write.table()` writes a data frame to a file and returns a `NULL` invisibly.


``` r
write.table(BOD, file = "BOD.csv")
```

It would have been more helpful if the function invisibly returned the actual object being written to the file, which could then be further used.


``` r
# cleanup
unlink("BOD.csv")
```

**Q3.** How does the `chdir` parameter of `source()` compare to `with_dir()`? Why might you prefer one to the other?

**A3.** The `chdir` parameter of `source()` is described as:

> if `TRUE` and `file` is a pathname, the `R` working directory is temporarily changed to the directory containing file for evaluating

That is, `chdir` allows changing working directory temporarily but *only* to the directory containing file being sourced:

While `withr::with_dir()` temporarily changes the current working directory:


``` r
withr::with_dir
#> function (new, code) 
#> {
#>     old <- setwd(dir = new)
#>     on.exit(setwd(old))
#>     force(code)
#> }
#> <bytecode: 0x56450b380668>
#> <environment: namespace:withr>
```

More importantly, its parameters `dir` allows temporarily changing working directory to *any* directory.

**Q4.** Write a function that opens a graphics device, runs the supplied code, and  closes the graphics device (always, regardless of whether or not the plotting code works).

**A4.** Here is a function that opens a graphics device, runs the supplied code, and  closes the graphics device:


``` r
with_png_device <- function(filename, code, ...) {
  grDevices::png(filename = filename, ...)
  on.exit(grDevices::dev.off(), add = TRUE)

  force(code)
}
```

**Q5.** We can use `on.exit()` to implement a simple version of `capture.output()`.


``` r
capture.output2 <- function(code) {
  temp <- tempfile()
  on.exit(file.remove(temp), add = TRUE, after = TRUE)

  sink(temp)
  on.exit(sink(), add = TRUE, after = TRUE)

  force(code)
  readLines(temp)
}

capture.output2(cat("a", "b", "c", sep = "\n"))
#> [1] "a" "b" "c"
```

Compare `capture.output()` to `capture.output2()`. How do the functions differ? What features have I removed to make the key ideas easier to see? How have I rewritten the key ideas so they're easier to understand?

**A5.** The `capture.output()` is significantly more complex, as can be seen by its definition:


``` r
capture.output
#> function (..., file = NULL, append = FALSE, type = c("output", 
#>     "message"), split = FALSE) 
#> {
#>     type <- match.arg(type)
#>     rval <- NULL
#>     closeit <- TRUE
#>     if (is.null(file)) 
#>         file <- textConnection("rval", "w", local = TRUE)
#>     else if (is.character(file)) 
#>         file <- file(file, if (append) 
#>             "a"
#>         else "w")
#>     else if (inherits(file, "connection")) {
#>         if (!isOpen(file)) 
#>             open(file, if (append) 
#>                 "a"
#>             else "w")
#>         else closeit <- FALSE
#>     }
#>     else stop("'file' must be NULL, a character string or a connection")
#>     sink(file, type = type, split = split)
#>     on.exit({
#>         sink(type = type, split = split)
#>         if (closeit) close(file)
#>     })
#>     for (i in seq_len(...length())) {
#>         out <- withVisible(...elt(i))
#>         if (out$visible) 
#>             print(out$value)
#>     }
#>     on.exit()
#>     sink(type = type, split = split)
#>     if (closeit) 
#>         close(file)
#>     rval %||% invisible(NULL)
#> }
#> <bytecode: 0x56450aa8e240>
#> <environment: namespace:utils>
```

Here are few key differences:

- `capture.output()` uses `print()` function to print to console:


``` r
capture.output(1)
#> [1] "[1] 1"

capture.output2(1)
#> character(0)
```

- `capture.output()` can capture messages as well:


``` r
capture.output(message("Hi there!"), "a", type = "message")
#> Hi there!
#> [1] "a"
#> character(0)
```

- `capture.output()` takes into account visibility of the expression:


``` r
capture.output(1, invisible(2), 3)
#> [1] "[1] 1" "[1] 3"
```

## Function forms (Exercises 6.8.6)

**Q1.** Rewrite the following code snippets into prefix form:


``` r
1 + 2 + 3

1 + (2 + 3)

if (length(x) <= 5) x[[5]] else x[[n]]
```

**A1.** Prefix forms for code snippets:


``` r
# The binary `+`  operator has left to right associative property.
`+`(`+`(1, 2), 3)

`+`(1, `(`(`+`(2, 3)))

`if`(cond = `<=`(length(x), 5), cons.expr = `[[`(x, 5), alt.expr = `[[`(x, n))
```

**Q2.**  Clarify the following list of odd function calls:


``` r
x <- sample(replace = TRUE, 20, x = c(1:10, NA))
y <- runif(min = 0, max = 1, 20)
cor(m = "k", y = y, u = "p", x = x)
```

**A2.** These functions don't have dots (`...`) as parameters, so the argument matching takes place in the following steps:

- exact matching for named arguments
- partial matching
- position-based

**Q3.** Explain why the following code fails:


``` r
modify(get("x"), 1) <- 10
#> Error: target of assignment expands to non-language object
```

**A3.** As provided in the book, the replacement function is defined as:


``` r
`modify<-` <- function(x, position, value) {
  x[position] <- value
  x
}
```

Let's re-write the provided code in prefix format to understand why it doesn't work:


``` r
get("x") <- `modify<-`(x = get("x"), position = 1, value = 10)
```

Although this works:


``` r
x <- 5
`modify<-`(x = get("x"), position = 1, value = 10)
#> [1] 10
```

The following doesn't because the code above evaluates to:


``` r
`get<-`("x", 10)
#> Error in `get<-`("x", 10): could not find function "get<-"
```

And there is no `get<-` function in R.

**Q4.** Create a replacement function that modifies a random location in a vector.

**A4.** A replacement function that modifies a random location in a vector:


``` r
`random_modify<-` <- function(x, value) {
  random_index <- sample(seq_along(x), size = 1)
  x[random_index] <- value
  return(x)
}
```

Let's try it out:


``` r
x1 <- rep("a", 10)
random_modify(x1) <- "X"
x1
#>  [1] "a" "a" "a" "a" "X" "a" "a" "a" "a" "a"

x2 <- rep("a", 10)
random_modify(x2) <- "Y"
x2
#>  [1] "a" "a" "a" "a" "a" "Y" "a" "a" "a" "a"

x3 <- rep(0, 15)
random_modify(x3) <- -4
x3
#>  [1]  0  0  0  0 -4  0  0  0  0  0  0  0  0  0  0

x4 <- rep(0, 15)
random_modify(x4) <- -1
x4
#>  [1]  0  0  0  0  0  0  0  0  0  0  0  0 -1  0  0
```

**Q5.** Write your own version of `+` that pastes its inputs together if they are character vectors but behaves as usual otherwise. In other words, make this code work:


``` r
1 + 2
#> [1] 3

"a" + "b"
#> [1] "ab"
```

**A5.** Infix operator to re-create the desired output:


``` r
`+` <- function(x, y) {
  if (is.character(x) || is.character(y)) {
    paste0(x, y)
  } else {
    base::`+`(x, y)
  }
}

1 + 2
#> [1] 3

"a" + "b"
#> [1] "ab"

rm("+", envir = .GlobalEnv)
```

**Q6.** Create a list of all the replacement functions found in the base package. Which ones are primitive functions? (Hint: use `apropos()`.)

**A6.** Replacement functions always have `<-` at the end of their names.

So, using `apropos()`, we can find all replacement functions in search paths and the filter out the ones that don't belong to `{base}` package:


``` r
ls_replacement <- apropos("<-", where = TRUE, mode = "function")

base_index <- which(grepl("base", searchpaths()))

ls_replacement <- ls_replacement[which(names(ls_replacement) == as.character(base_index))]

unname(ls_replacement)
#>  [1] ".rowNamesDF<-"           "[[<-"                   
#>  [3] "[[<-.data.frame"         "[[<-.factor"            
#>  [5] "[[<-.numeric_version"    "[[<-.POSIXlt"           
#>  [7] "[<-"                     "[<-.data.frame"         
#>  [9] "[<-.Date"                "[<-.difftime"           
#> [11] "[<-.factor"              "[<-.numeric_version"    
#> [13] "[<-.POSIXct"             "[<-.POSIXlt"            
#> [15] "@<-"                     "<-"                     
#> [17] "<<-"                     "$<-"                    
#> [19] "$<-.data.frame"          "$<-.POSIXlt"            
#> [21] "attr<-"                  "attributes<-"           
#> [23] "body<-"                  "class<-"                
#> [25] "colnames<-"              "comment<-"              
#> [27] "diag<-"                  "dim<-"                  
#> [29] "dimnames<-"              "dimnames<-.data.frame"  
#> [31] "Encoding<-"              "environment<-"          
#> [33] "formals<-"               "is.na<-"                
#> [35] "is.na<-.default"         "is.na<-.factor"         
#> [37] "is.na<-.numeric_version" "length<-"               
#> [39] "length<-.Date"           "length<-.difftime"      
#> [41] "length<-.factor"         "length<-.POSIXct"       
#> [43] "length<-.POSIXlt"        "levels<-"               
#> [45] "levels<-.factor"         "mode<-"                 
#> [47] "mostattributes<-"        "names<-"                
#> [49] "names<-.POSIXlt"         "oldClass<-"             
#> [51] "parent.env<-"            "regmatches<-"           
#> [53] "row.names<-"             "row.names<-.data.frame" 
#> [55] "row.names<-.default"     "rownames<-"             
#> [57] "split<-"                 "split<-.data.frame"     
#> [59] "split<-.default"         "storage.mode<-"         
#> [61] "substr<-"                "substring<-"            
#> [63] "units<-"                 "units<-.difftime"
```

The primitive replacement functions can be listed using `is.primitive()`:


``` r
mget(ls_replacement, envir = baseenv()) %>%
  purrr::keep(is.primitive) %>%
  names()
#>  [1] "[[<-"           "[<-"            "@<-"           
#>  [4] "<-"             "<<-"            "$<-"           
#>  [7] "attr<-"         "attributes<-"   "class<-"       
#> [10] "dim<-"          "dimnames<-"     "environment<-" 
#> [13] "length<-"       "levels<-"       "names<-"       
#> [16] "oldClass<-"     "storage.mode<-"
```

**Q7.** What are valid names for user-created infix functions?

**A7.**  As mentioned in the respective [section](https://adv-r.hadley.nz/functions.html#infix-functions) of the book:

> The names of infix functions are more flexible than regular R functions: they can contain any sequence of characters except for `%`.

**Q8.** Create an infix `xor()` operator.

**A8.** Exclusive OR  is a logical operation that is `TRUE` if and only if its arguments differ (one is `TRUE`, the other is `FALSE`).


``` r
lv1 <- c(TRUE, FALSE, TRUE, FALSE)
lv2 <- c(TRUE, TRUE, FALSE, FALSE)

xor(lv1, lv2)
#> [1] FALSE  TRUE  TRUE FALSE
```

We can create infix operator for exclusive OR like so:


``` r
`%xor%` <- function(x, y) {
  !((x & y) | !(x | y))
}

lv1 %xor% lv2
#> [1] FALSE  TRUE  TRUE FALSE

TRUE %xor% TRUE
#> [1] FALSE
```

The function is vectorized over its inputs because the underlying logical operators themselves are vectorized.

**Q9.** Create infix versions of the set functions `intersect()`, `union()`, and `setdiff()`. You might call them `%n%`, `%u%`, and `%/%` to match conventions from mathematics.

**A9.** The required infix operators can be created as following:


``` r
`%n%` <- function(x, y) {
  intersect(x, y)
}

`%u%` <- function(x, y) {
  union(x, y)
}

`%/%` <- function(x, y) {
  setdiff(x, y)
}
```

We can check that the outputs agree with the underlying functions:


``` r
(x <- c(sort(sample(1:20, 9)), NA))
#>  [1]  4  7  8  9 11 13 15 16 20 NA
(y <- c(sort(sample(3:23, 7)), NA))
#> [1]  9 10 13 15 17 19 20 NA

identical(intersect(x, y), x %n% y)
#> [1] TRUE
identical(union(x, y), x %u% y)
#> [1] TRUE
identical(setdiff(x, y), x %/% y)
#> [1] TRUE
```

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
#>  date     2025-03-03
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
#>  datasets    * 4.4.3   2025-02-28 [3] local
#>  digest        0.6.37  2024-08-19 [1] RSPM
#>  dplyr       * 1.1.4   2023-11-17 [1] RSPM
#>  emoji         16.0.0  2024-10-28 [1] RSPM
#>  evaluate      1.0.3   2025-01-10 [1] RSPM
#>  fastmap       1.2.0   2024-05-15 [1] RSPM
#>  forcats     * 1.0.0   2023-01-29 [1] RSPM
#>  generics      0.1.3   2022-07-05 [1] RSPM
#>  ggplot2     * 3.5.1   2024-04-23 [1] RSPM
#>  glue          1.8.0   2024-09-30 [1] RSPM
#>  graphics    * 4.4.3   2025-02-28 [3] local
#>  grDevices   * 4.4.3   2025-02-28 [3] local
#>  grid          4.4.3   2025-02-28 [3] local
#>  gtable        0.3.6   2024-10-25 [1] RSPM
#>  hms           1.1.3   2023-03-21 [1] RSPM
#>  htmltools     0.5.8.1 2024-04-04 [1] RSPM
#>  knitr         1.49    2024-11-08 [1] RSPM
#>  lifecycle     1.0.4   2023-11-07 [1] RSPM
#>  lobstr        1.1.2   2022-06-22 [1] RSPM
#>  lubridate   * 1.9.4   2024-12-08 [1] RSPM
#>  magrittr    * 2.0.3   2022-03-30 [1] RSPM
#>  methods     * 4.4.3   2025-02-28 [3] local
#>  munsell       0.5.1   2024-04-01 [1] RSPM
#>  pillar        1.10.1  2025-01-07 [1] RSPM
#>  pkgconfig     2.0.3   2019-09-22 [1] RSPM
#>  purrr       * 1.0.4   2025-02-05 [1] RSPM
#>  R6            2.6.1   2025-02-15 [1] RSPM
#>  readr       * 2.1.5   2024-01-10 [1] RSPM
#>  rlang         1.1.5   2025-01-17 [1] RSPM
#>  rmarkdown     2.29    2024-11-04 [1] RSPM
#>  scales        1.3.0   2023-11-28 [1] RSPM
#>  sessioninfo   1.2.3   2025-02-05 [1] RSPM
#>  stats       * 4.4.3   2025-02-28 [3] local
#>  stringi       1.8.4   2024-05-06 [1] RSPM
#>  stringr     * 1.5.1   2023-11-14 [1] RSPM
#>  tibble      * 3.2.1   2023-03-20 [1] RSPM
#>  tidyr       * 1.3.1   2024-01-24 [1] RSPM
#>  tidyselect    1.2.1   2024-03-11 [1] RSPM
#>  tidyverse   * 2.0.0   2023-02-22 [1] RSPM
#>  timechange    0.3.0   2024-01-18 [1] RSPM
#>  tools         4.4.3   2025-02-28 [3] local
#>  tzdb          0.4.0   2023-05-12 [1] RSPM
#>  utf8          1.2.4   2023-10-22 [1] RSPM
#>  utils       * 4.4.3   2025-02-28 [3] local
#>  vctrs         0.6.5   2023-12-01 [1] RSPM
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

