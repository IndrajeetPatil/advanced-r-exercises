# Rewriting R code in C++




``` r
library(Rcpp, warn.conflicts = FALSE)
```

## Getting started with C++ (Exercises 25.2.6)

**Q1.** With the basics of C++ in hand, it's now a great time to practice by reading and writing some simple C++ functions. For each of the following functions, read the code and figure out what the corresponding base R function is. You might not understand every part of the code yet, but you should be able to figure out the basics of what the function does.


``` cpp
#include <Rcpp.h>
using namespace Rcpp;

// [[Rcpp::export]]
double f1(NumericVector x) {
  int n = x.size();
  double y = 0;

  for(int i = 0; i < n; ++i) {
    y += x[i] / n;
  }
  return y;
}

// [[Rcpp::export]]
NumericVector f2(NumericVector x) {
  int n = x.size();
  NumericVector out(n);

  out[0] = x[0];
  for(int i = 1; i < n; ++i) {
    out[i] = out[i - 1] + x[i];
  }
  return out;
}

// [[Rcpp::export]]
bool f3(LogicalVector x) {
  int n = x.size();

  for(int i = 0; i < n; ++i) {
    if (x[i]) return true;
  }
  return false;
}

// [[Rcpp::export]]
int f4(Function pred, List x) {
  int n = x.size();

  for(int i = 0; i < n; ++i) {
    LogicalVector res = pred(x[i]);
    if (res[0]) return i + 1;
  }
  return 0;
}

// [[Rcpp::export]]
NumericVector f5(NumericVector x, NumericVector y) {
  int n = std::max(x.size(), y.size());
  NumericVector x1 = rep_len(x, n);
  NumericVector y1 = rep_len(y, n);

  NumericVector out(n);

  for (int i = 0; i < n; ++i) {
    out[i] = std::min(x1[i], y1[i]);
  }

  return out;
}
```

**A1.** 

`f1()` is the same as `mean()`:


``` r
x <- c(1, 2, 3, 4, 5, 6)

f1(x)
#> [1] 3.5
mean(x)
#> [1] 3.5
```

`f2()` is the same as `cumsum()`:


``` r
x <- c(1, 3, 5, 6)

f2(x)
#> [1]  1  4  9 15
cumsum(x)
#> [1]  1  4  9 15
```

`f3()` is the same as `any()`:


``` r
x1 <- c(TRUE, FALSE, FALSE, TRUE)
x2 <- c(FALSE, FALSE)

f3(x1)
#> [1] TRUE
any(x1)
#> [1] TRUE

f3(x2)
#> [1] FALSE
any(x2)
#> [1] FALSE
```

`f4()` is the same as `Position()`:


``` r
x <- list("a", TRUE, "m", 2)

f4(is.numeric, x)
#> [1] 4
Position(is.numeric, x)
#> [1] 4
```

`f5()` is the same as `pmin()`:


``` r
v1 <- c(1, 3, 4, 5, 6, 7)
v2 <- c(1, 2, 7, 2, 8, 1)

f5(v1, v2)
#> [1] 1 2 4 2 6 1
pmin(v1, v2)
#> [1] 1 2 4 2 6 1
```

**Q2.** To practice your function writing skills, convert the following functions into C++. For now, assume the inputs have no missing values.

1. `all()`.

2. `cumprod()`, `cummin()`, `cummax()`.

3. `diff()`. Start by assuming lag 1, and then generalise for lag `n`.

4. `range()`.

5. `var()`. Read about the approaches you can take on  [Wikipedia](http://en.wikipedia.org/wiki/Algorithms_for_calculating_variance). Whenever implementing a numerical algorithm, it's always good to check what is already known about the problem.

**A2.** The performance benefits are not going to be observed if the function is primitive since those are already tuned to the max in R for performance. So, expect performance gain only for `diff()` and `var()`.


``` r
is.primitive(all)
#> [1] TRUE
is.primitive(cumprod)
#> [1] TRUE
is.primitive(diff)
#> [1] FALSE
is.primitive(range)
#> [1] TRUE
is.primitive(var)
#> [1] FALSE
```

- `all()`


``` cpp
#include <vector>
// [[Rcpp::plugins(cpp11)]]

// [[Rcpp::export]]
bool allC(std::vector<bool> x)
{
    for (const auto& xElement : x)
    {
        if (!xElement) return false;
    }

    return true;
}
```


``` r
v1 <- rep(TRUE, 10)
v2 <- c(rep(TRUE, 5), rep(FALSE, 5))

all(v1)
#> [1] TRUE
allC(v1)
#> [1] TRUE

all(v2)
#> [1] FALSE
allC(v2)
#> [1] FALSE

# performance benefits?
bench::mark(
  all(c(rep(TRUE, 1000), rep(FALSE, 1000))),
  allC(c(rep(TRUE, 1000), rep(FALSE, 1000))),
  iterations = 100
)
#> # A tibble: 2 × 6
#>   expression                                      min
#>   <bch:expr>                                 <bch:tm>
#> 1 all(c(rep(TRUE, 1000), rep(FALSE, 1000)))    5.95µs
#> 2 allC(c(rep(TRUE, 1000), rep(FALSE, 1000)))   7.88µs
#>     median `itr/sec` mem_alloc `gc/sec`
#>   <bch:tm>     <dbl> <bch:byt>    <dbl>
#> 1   6.57µs   118875.    15.8KB        0
#> 2   8.18µs   116109.    15.8KB        0
```

- `cumprod()`


``` cpp
#include <vector>

// [[Rcpp::export]]
std::vector<double> cumprodC(const std::vector<double> &x)
{
    std::vector<double> out{x};

    for (std::size_t i = 1; i < x.size(); i++)
    {
        out[i] = out[i - 1] * x[i];
    }

    return out;
}
```



``` r
v1 <- c(10, 4, 6, 8)

cumprod(v1)
#> [1]   10   40  240 1920
cumprodC(v1)
#> [1]   10   40  240 1920

# performance benefits?
bench::mark(
  cumprod(v1),
  cumprodC(v1),
  iterations = 100
)
#> # A tibble: 2 × 6
#>   expression        min   median `itr/sec` mem_alloc
#>   <bch:expr>   <bch:tm> <bch:tm>     <dbl> <bch:byt>
#> 1 cumprod(v1)      90ns    110ns  6560507.        0B
#> 2 cumprodC(v1)    682ns    772ns   965265.    4.12KB
#>   `gc/sec`
#>      <dbl>
#> 1        0
#> 2        0
```

- `cumminC()`


``` cpp
#include <vector>
// [[Rcpp::plugins(cpp11)]]

// [[Rcpp::export]]
std::vector<double> cumminC(const std::vector<double> &x)
{
    std::vector<double> out{x};

    for (std::size_t i = 1; i < x.size(); i++)
    {
        out[i] = (out[i] < out[i - 1]) ? out[i] : out[i - 1];
    }

    return out;
}
```


``` r
v1 <- c(3:1, 2:0, 4:2)

cummin(v1)
#> [1] 3 2 1 1 1 0 0 0 0
cumminC(v1)
#> [1] 3 2 1 1 1 0 0 0 0

# performance benefits?
bench::mark(
  cummin(v1),
  cumminC(v1),
  iterations = 100
)
#> # A tibble: 2 × 6
#>   expression       min   median `itr/sec` mem_alloc `gc/sec`
#>   <bch:expr>  <bch:tm> <bch:tm>     <dbl> <bch:byt>    <dbl>
#> 1 cummin(v1)     110ns    121ns  6757077.        0B        0
#> 2 cumminC(v1)    741ns    777ns   843569.    4.12KB        0
```

- `cummaxC()`


``` cpp
#include <vector>
// [[Rcpp::plugins(cpp11)]]

// [[Rcpp::export]]
std::vector<double> cummaxC(const std::vector<double> &x)
{
    std::vector<double> out{x};

    for (std::size_t i = 1; i < x.size(); i++)
    {
        out[i] = (out[i] > out[i - 1]) ? out[i] : out[i - 1];
    }
    
    return out;
}
```


``` r
v1 <- c(3:1, 2:0, 4:2)

cummax(v1)
#> [1] 3 3 3 3 3 3 4 4 4
cummaxC(v1)
#> [1] 3 3 3 3 3 3 4 4 4

# performance benefits?
bench::mark(
  cummax(v1),
  cummaxC(v1),
  iterations = 100
)
#> # A tibble: 2 × 6
#>   expression       min   median `itr/sec` mem_alloc `gc/sec`
#>   <bch:expr>  <bch:tm> <bch:tm>     <dbl> <bch:byt>    <dbl>
#> 1 cummax(v1)     100ns    190ns  4874220.        0B        0
#> 2 cummaxC(v1)    752ns    912ns  1036955.    4.12KB        0
```

- `diff()`


``` cpp
#include <vector>
#include <functional>
#include <algorithm>
using namespace std;
// [[Rcpp::plugins(cpp11)]]

// [[Rcpp::export]]
std::vector<double> diffC(const std::vector<double> &x, int lag)
{
    std::vector<double> vec_start;
    std::vector<double> vec_lagged;
    std::vector<double> vec_diff;

    for (std::size_t i = lag; i < x.size(); i++)
    {
        vec_lagged.push_back(x[i]);
    }

    for (std::size_t i = 0; i < (x.size() - lag); i++)
    {
        vec_start.push_back(x[i]);
    }

    std::transform(
        vec_lagged.begin(), vec_lagged.end(),
        vec_start.begin(), std::back_inserter(vec_diff),
        std::minus<double>());

    return vec_diff;
}
```


``` r
v1 <- c(1, 2, 4, 8, 13)
v2 <- c(1, 2, NA, 8, 13)

diff(v1, 2)
#> [1] 3 6 9
diffC(v1, 2)
#> [1] 3 6 9

diff(v2, 2)
#> [1] NA  6 NA
diffC(v2, 2)
#> [1] NA  6 NA

# performance benefits?
bench::mark(
  diff(v1, 2),
  diffC(v1, 2),
  iterations = 100
)
#> # A tibble: 2 × 6
#>   expression        min   median `itr/sec` mem_alloc
#>   <bch:expr>   <bch:tm> <bch:tm>     <dbl> <bch:byt>
#> 1 diff(v1, 2)    4.02µs   4.31µs   209833.        0B
#> 2 diffC(v1, 2)   1.05µs    1.1µs   829849.        0B
#>   `gc/sec`
#>      <dbl>
#> 1        0
#> 2        0
```

- `range()`


``` cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

// [[Rcpp::export]]
std::vector<double> rangeC(std::vector<double> x)
{
    std::vector<double> rangeVec{0.0, 0.0};

    rangeVec.at(0) = *std::min_element(x.begin(), x.end());
    rangeVec.at(1) = *std::max_element(x.begin(), x.end());

    return rangeVec;
}
```


``` r
v1 <- c(10, 4, 6, 8)

range(v1)
#> [1]  4 10
rangeC(v1)
#> [1]  4 10

# performance benefits?
bench::mark(
  range(v1),
  rangeC(v1),
  iterations = 100
)
#> # A tibble: 2 × 6
#>   expression      min   median `itr/sec` mem_alloc `gc/sec`
#>   <bch:expr> <bch:tm> <bch:tm>     <dbl> <bch:byt>    <dbl>
#> 1 range(v1)    2.42µs   2.58µs   342922.        0B        0
#> 2 rangeC(v1) 701.05ns 802.04ns  1165964.    4.12KB        0
```

- `var()`


``` cpp
#include <vector>
#include <cmath>
#include <numeric>
using namespace std;
// [[Rcpp::plugins(cpp11)]]

// [[Rcpp::export]]
double variance(std::vector<double> x)
{
    double sumSquared{0};

    double mean = std::accumulate(x.begin(), x.end(), 0.0) / x.size();

    for (const auto& xElement : x)
    {
        sumSquared += pow(xElement - mean, 2.0);
    }

    return sumSquared / (x.size() - 1);
}
```



``` r
v1 <- c(1, 4, 7, 8)

var(v1)
#> [1] 10
variance(v1)
#> [1] 10

# performance benefits?
bench::mark(
  var(v1),
  variance(v1),
  iterations = 100
)
#> # A tibble: 2 × 6
#>   expression        min   median `itr/sec` mem_alloc
#>   <bch:expr>   <bch:tm> <bch:tm>     <dbl> <bch:byt>
#> 1 var(v1)        5.29µs   5.84µs   165438.        0B
#> 2 variance(v1) 661.01ns 692.09ns  1284336.    4.12KB
#>   `gc/sec`
#>      <dbl>
#> 1        0
#> 2        0
```

## Missing values (Exercises 25.4.5)

**Q1.** Rewrite any of the functions from Exercise 25.2.6 to deal with missing values. If `na.rm` is true, ignore the missing values. If `na.rm` is false,  return a missing value if the input contains any missing values. Some good functions to practice with are `min()`, `max()`, `range()`, `mean()`, and `var()`.

**A1.** We will only create a version of `range()` that deals with missing values. The same principle applies to others:


``` cpp
#include <iostream>
#include <vector>
#include <algorithm>
#include <math.h>
#include <Rcpp.h>
using namespace std;
// [[Rcpp::plugins(cpp11)]]

// [[Rcpp::export]]
std::vector<double> rangeC_NA(std::vector<double> x, bool removeNA = true)
{
    std::vector<double> rangeVec{0.0, 0.0};

    bool naPresent = std::any_of(
        x.begin(),
        x.end(),
        [](double d)
        { return isnan(d); });

    if (naPresent)
    {
        if (removeNA)
        {
            std::remove(x.begin(), x.end(), NAN);
        }
        else
        {
            rangeVec.at(0) = NA_REAL; // NAN;
            rangeVec.at(1) = NA_REAL; // NAN;

            return rangeVec;
        }
    }

    rangeVec.at(0) = *std::min_element(x.begin(), x.end());
    rangeVec.at(1) = *std::max_element(x.begin(), x.end());

    return rangeVec;
}
```


``` r
v1 <- c(10, 4, NA, 6, 8)

range(v1, na.rm = FALSE)
#> [1] NA NA
rangeC_NA(v1, FALSE)
#> [1] NA NA

range(v1, na.rm = TRUE)
#> [1]  4 10
rangeC_NA(v1, TRUE)
#> [1]  4 10
```

**Q2.** Rewrite `cumsum()` and `diff()` so they can handle missing values. Note that these functions have slightly more complicated behaviour.

**A2.** The `cumsum()` docs say:

> An `NA` value in `x` causes the corresponding and following elements of the return value to be `NA`, as does integer overflow in cumsum (with a warning).

Similarly, `diff()` docs say:

> `NA`'s propagate.

Therefore, both of these functions don't allow removing missing values and the `NA`s propagate.

As seen from the examples above, `diffC()` already behaves this way.

Similarly, `cumsumC()` propagates `NA`s as well.


``` cpp
#include <Rcpp.h>
using namespace Rcpp;
// [[Rcpp::plugins(cpp11)]]

// [[Rcpp::export]]
NumericVector cumsumC(NumericVector x) {
  int n = x.size();
  NumericVector out(n);
  
  out[0] = x[0];
  for(int i = 1; i < n; ++i) {
    out[i] = out[i - 1] + x[i];
  }
  
  return out;
}
```


``` r
v1 <- c(1, 2, 3, 4)
v2 <- c(1, 2, NA, 4)

cumsum(v1)
#> [1]  1  3  6 10
cumsumC(v1)
#> [1]  1  3  6 10

cumsum(v2)
#> [1]  1  3 NA NA
cumsumC(v2)
#> [1]  1  3 NA NA
```

## Standard Template Library (Exercises 25.5.7)

**Q1.** To practice using the STL algorithms and data structures, implement the following using R functions in C++, using the hints provided:

**A1.** 

1. `median.default()` using `partial_sort`.


``` cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;
// [[Rcpp::plugins(cpp11)]]

// [[Rcpp::export]]
double medianC(std::vector<double> &x)
{
    int middleIndex = static_cast<int>(x.size() / 2);

    std::partial_sort(x.begin(), x.begin() + middleIndex, x.end());

    // for even number of observations
    if (x.size() % 2 == 0)
    {
        return (x[middleIndex - 1] + x[middleIndex]) / 2;
    }

    return x[middleIndex];
}
```



``` r
v1 <- c(1, 3, 3, 6, 7, 8, 9)
v2 <- c(1, 2, 3, 4, 5, 6, 8, 9)

median.default(v1)
#> [1] 6
medianC(v1)
#> [1] 6

median.default(v2)
#> [1] 4.5
medianC(v2)
#> [1] 4.5

# performance benefits?
bench::mark(
  median.default(v2),
  medianC(v2),
  iterations = 100
)
#> # A tibble: 2 × 6
#>   expression              min   median `itr/sec` mem_alloc
#>   <bch:expr>         <bch:tm> <bch:tm>     <dbl> <bch:byt>
#> 1 median.default(v2)   20.8µs   22.1µs    42816.        0B
#> 2 medianC(v2)         701.1ns  782.1ns  1081602.        0B
#>   `gc/sec`
#>      <dbl>
#> 1        0
#> 2        0
```

1. `%in%` using `unordered_set` and the `find()` or `count()` methods.


``` cpp
#include <vector>
#include <unordered_set>
using namespace std;
// [[Rcpp::plugins(cpp11)]]

// [[Rcpp::export]]
std::vector<bool> matchC(const std::vector<double> &x, const std::vector<double> &table)
{
    std::unordered_set<double> tableUnique(table.begin(), table.end());
    std::vector<bool> out;

    for (const auto &xElem : x)
    {
        out.push_back(tableUnique.find(xElem) != tableUnique.end() ? true : false);
    }

    return out;
}
```


``` r
x1 <- c(3, 4, 8)
x2 <- c(1, 2, 3, 3, 4, 4, 5, 6)

x1 %in% x2
#> [1]  TRUE  TRUE FALSE
matchC(x1, x2)
#> [1]  TRUE  TRUE FALSE

# performance benefits?
bench::mark(
  x1 %in% x2,
  matchC(x1, x2),
  iterations = 100
)
#> # A tibble: 2 × 6
#>   expression          min   median `itr/sec` mem_alloc
#>   <bch:expr>     <bch:tm> <bch:tm>     <dbl> <bch:byt>
#> 1 x1 %in% x2     871.14ns 982.08ns   962904.        0B
#> 2 matchC(x1, x2)   1.23µs   1.31µs   716267.    4.12KB
#>   `gc/sec`
#>      <dbl>
#> 1        0
#> 2        0
```

1. `unique()` using an `unordered_set` (challenge: do it in one line!).


``` cpp
#include <unordered_set>
#include <vector>
#include <iostream>
using namespace std;
// [[Rcpp::plugins(cpp11)]]

// [[Rcpp::export]]
std::unordered_set<double> uniqueC(const std::vector<double> &x)
{
    std::unordered_set<double> xSet(x.begin(), x.end());

    return xSet;
}
```

Note that these functions are **not** comparable. As far as I can see, there is no way to get the same output as the R version of the function using the `unordered_set` data structure.


``` r
v1 <- c(1, 3, 3, 6, 7, 8, 9)

unique(v1)
#> [1] 1 3 6 7 8 9
uniqueC(v1)
#> [1] 9 8 7 6 3 1
```

We can make comparable version using `set` data structure:


``` cpp
#include <set>
#include <vector>
#include <iostream>
using namespace std;
// [[Rcpp::plugins(cpp11)]]

// [[Rcpp::export]]
std::set<double> uniqueC2(const std::vector<double> &x)
{
    std::set<double> xSet(x.begin(), x.end());

    return xSet;
}
```


``` r
v1 <- c(1, 3, 3, 6, 7, 8, 9)

unique(v1)
#> [1] 1 3 6 7 8 9
uniqueC2(v1)
#> [1] 1 3 6 7 8 9

# performance benefits?
bench::mark(
  unique(v1),
  uniqueC2(v1),
  iterations = 100
)
#> # A tibble: 2 × 6
#>   expression        min   median `itr/sec` mem_alloc
#>   <bch:expr>   <bch:tm> <bch:tm>     <dbl> <bch:byt>
#> 1 unique(v1)     2.09µs   2.36µs   366753.        0B
#> 2 uniqueC2(v1) 831.09ns 912.11ns   964058.    4.12KB
#>   `gc/sec`
#>      <dbl>
#> 1        0
#> 2        0
```

1. `min()` using `std::min()`, or `max()` using `std::max()`.


``` cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;
// [[Rcpp::plugins(cpp11)]]

// [[Rcpp::export]]
const double minC(const std::vector<double> &x)
{
     return *std::min_element(x.begin(), x.end());
}

// [[Rcpp::export]]
const double maxC(std::vector<double> x)
{
     return *std::max_element(x.begin(), x.end());
}
```


``` r
v1 <- c(3, 3, 6, 1, 9, 7, 8)

min(v1)
#> [1] 1
minC(v1)
#> [1] 1

# performance benefits?
bench::mark(
  min(v1),
  minC(v1),
  iterations = 100
)
#> # A tibble: 2 × 6
#>   expression      min   median `itr/sec` mem_alloc `gc/sec`
#>   <bch:expr> <bch:tm> <bch:tm>     <dbl> <bch:byt>    <dbl>
#> 1 min(v1)       211ns    231ns  3647051.        0B        0
#> 2 minC(v1)      661ns    691ns  1325458.    4.12KB        0

max(v1)
#> [1] 9
maxC(v1)
#> [1] 9

# performance benefits?
bench::mark(
  max(v1),
  maxC(v1),
  iterations = 100
)
#> # A tibble: 2 × 6
#>   expression      min   median `itr/sec` mem_alloc `gc/sec`
#>   <bch:expr> <bch:tm> <bch:tm>     <dbl> <bch:byt>    <dbl>
#> 1 max(v1)       210ns    230ns  3999616.        0B        0
#> 2 maxC(v1)      651ns    677ns  1349210.    4.12KB        0
```

1. `which.min()` using `min_element`, or `which.max()` using `max_element`.


``` cpp
#include <vector>
#include <algorithm>
using namespace std;
// [[Rcpp::plugins(cpp11)]]

// [[Rcpp::export]]
int which_maxC(std::vector<double> &x)
{
    int maxIndex = std::distance(x.begin(), std::max_element(x.begin(), x.end()));
  
    // R is 1-index based, while C++ is 0-index based
    return maxIndex + 1;
}

// [[Rcpp::export]]
int which_minC(std::vector<double> &x)
{
    int minIndex = std::distance(x.begin(), std::min_element(x.begin(), x.end()));
  
    // R is 1-index based, while C++ is 0-index based
    return minIndex + 1;
}
```



``` r
v1 <- c(3, 3, 6, 1, 9, 7, 8)

which.min(v1)
#> [1] 4
which_minC(v1)
#> [1] 4

# performance benefits?
bench::mark(
  which.min(v1),
  which_minC(v1),
  iterations = 100
)
#> # A tibble: 2 × 6
#>   expression          min   median `itr/sec` mem_alloc
#>   <bch:expr>     <bch:tm> <bch:tm>     <dbl> <bch:byt>
#> 1 which.min(v1)     370ns    391ns  2161561.        0B
#> 2 which_minC(v1)    661ns    701ns  1279334.    4.12KB
#>   `gc/sec`
#>      <dbl>
#> 1        0
#> 2        0

which.max(v1)
#> [1] 5
which_maxC(v1)
#> [1] 5

# performance benefits?
bench::mark(
  which.max(v1),
  which_maxC(v1),
  iterations = 100
)
#> # A tibble: 2 × 6
#>   expression          min   median `itr/sec` mem_alloc
#>   <bch:expr>     <bch:tm> <bch:tm>     <dbl> <bch:byt>
#> 1 which.max(v1)     391ns    411ns  1596900.        0B
#> 2 which_maxC(v1)    671ns    702ns  1290289.    4.12KB
#>   `gc/sec`
#>      <dbl>
#> 1        0
#> 2        0
```

1. `setdiff()`, `union()`, and `intersect()` for integers using sorted ranges and `set_union`, `set_intersection` and `set_difference`.

Note that the following C++ implementations of given functions are not strictly equivalent to their R versions. As far as I can see, there is no way for them to be identical while satisfying the specifications mentioned in the question.

- `union()`


``` cpp
#include <algorithm>
#include <iostream>
#include <vector>
#include <set>
using namespace std;
// [[Rcpp::plugins(cpp11)]]

// [[Rcpp::export]]
std::set<int> unionC(std::vector<int> &v1, std::vector<int> &v2)
{
    std::sort(v1.begin(), v1.end());
    std::sort(v2.begin(), v2.end());

    std::vector<int> union_vec(v1.size() + v2.size());
    auto it = std::set_union(v1.begin(), v1.end(), v2.begin(), v2.end(), union_vec.begin());

    union_vec.resize(it - union_vec.begin());
    std::set<int> union_set(union_vec.begin(), union_vec.end());

    return union_set;
}
```


``` r
v1 <- c(1, 4, 5, 5, 5, 6, 2)
v2 <- c(4, 1, 6, 8)

union(v1, v2)
#> [1] 1 4 5 6 2 8
unionC(v1, v2)
#> [1] 1 2 4 5 6 8
```

- `intersect()`


``` cpp
#include <algorithm>
#include <iostream>
#include <vector>
#include <set>
using namespace std;
// [[Rcpp::plugins(cpp11)]]

// [[Rcpp::export]]
std::set<int> intersectC(std::vector<int> &v1, std::vector<int> &v2)
{
    std::sort(v1.begin(), v1.end());
    std::sort(v2.begin(), v2.end());

    std::vector<int> union_vec(v1.size() + v2.size());
    auto it = std::set_intersection(v1.begin(), v1.end(), v2.begin(), v2.end(), union_vec.begin());

    union_vec.resize(it - union_vec.begin());
    std::set<int> union_set(union_vec.begin(), union_vec.end());

    return union_set;
}
```


``` r
v1 <- c(1, 4, 5, 5, 5, 6, 2)
v2 <- c(4, 1, 6, 8)

intersect(v1, v2)
#> [1] 1 4 6
intersectC(v1, v2)
#> [1] 1 4 6
```

- `setdiff()`


``` cpp
#include <algorithm>
#include <iostream>
#include <vector>
#include <set>
using namespace std;
// [[Rcpp::plugins(cpp11)]]

// [[Rcpp::export]]
std::set<int> setdiffC(std::vector<int> &v1, std::vector<int> &v2)
{
    std::sort(v1.begin(), v1.end());
    std::sort(v2.begin(), v2.end());

    std::vector<int> union_vec(v1.size() + v2.size());
    auto it = std::set_difference(v1.begin(), v1.end(), v2.begin(), v2.end(), union_vec.begin());

    union_vec.resize(it - union_vec.begin());
    std::set<int> union_set(union_vec.begin(), union_vec.end());

    return union_set;
}
```


``` r
v1 <- c(1, 4, 5, 5, 5, 6, 2)
v2 <- c(4, 1, 6, 8)

setdiff(v1, v2)
#> [1] 5 2
setdiffC(v1, v2)
#> [1] 2 5
```

## Session information


``` r
sessioninfo::session_info(include_base = TRUE)
#> ─ Session info ───────────────────────────────────────────
#>  setting  value
#>  version  R version 4.5.2 (2025-10-31)
#>  os       Ubuntu 24.04.3 LTS
#>  system   x86_64, linux-gnu
#>  ui       X11
#>  language (EN)
#>  collate  C.UTF-8
#>  ctype    C.UTF-8
#>  tz       UTC
#>  date     2026-01-13
#>  pandoc   3.8.3 @ /opt/hostedtoolcache/pandoc/3.8.3/x64/ (via rmarkdown)
#>  quarto   NA
#> 
#> ─ Packages ───────────────────────────────────────────────
#>  package     * version date (UTC) lib source
#>  base        * 4.5.2   2025-10-31 [3] local
#>  bench         1.1.4   2025-01-16 [1] RSPM
#>  bookdown      0.46    2025-12-05 [1] RSPM
#>  cli           3.6.5   2025-04-23 [1] RSPM
#>  compiler      4.5.2   2025-10-31 [3] local
#>  datasets    * 4.5.2   2025-10-31 [3] local
#>  digest        0.6.39  2025-11-19 [1] RSPM
#>  emoji         16.0.0  2024-10-28 [1] RSPM
#>  evaluate      1.0.5   2025-08-27 [1] RSPM
#>  fastmap       1.2.0   2024-05-15 [1] RSPM
#>  glue          1.8.0   2024-09-30 [1] RSPM
#>  graphics    * 4.5.2   2025-10-31 [3] local
#>  grDevices   * 4.5.2   2025-10-31 [3] local
#>  htmltools     0.5.9   2025-12-04 [1] RSPM
#>  knitr         1.51    2025-12-20 [1] RSPM
#>  lifecycle     1.0.5   2026-01-08 [1] RSPM
#>  magrittr    * 2.0.4   2025-09-12 [1] RSPM
#>  methods     * 4.5.2   2025-10-31 [3] local
#>  pillar        1.11.1  2025-09-17 [1] RSPM
#>  pkgconfig     2.0.3   2019-09-22 [1] RSPM
#>  profmem       0.7.0   2025-05-02 [1] RSPM
#>  Rcpp        * 1.1.1   2026-01-10 [1] RSPM
#>  rlang         1.1.7   2026-01-09 [1] RSPM
#>  rmarkdown     2.30    2025-09-28 [1] RSPM
#>  sessioninfo   1.2.3   2025-02-05 [1] RSPM
#>  stats       * 4.5.2   2025-10-31 [3] local
#>  stringi       1.8.7   2025-03-27 [1] RSPM
#>  stringr       1.6.0   2025-11-04 [1] RSPM
#>  tibble        3.3.1   2026-01-11 [1] RSPM
#>  tools         4.5.2   2025-10-31 [3] local
#>  utf8          1.2.6   2025-06-08 [1] RSPM
#>  utils       * 4.5.2   2025-10-31 [3] local
#>  vctrs         0.6.5   2023-12-01 [1] RSPM
#>  xfun          0.55    2025-12-16 [1] RSPM
#>  yaml          2.3.12  2025-12-10 [1] RSPM
#> 
#>  [1] /home/runner/work/_temp/Library
#>  [2] /opt/R/4.5.2/lib/R/site-library
#>  [3] /opt/R/4.5.2/lib/R/library
#>  * ── Packages attached to the search path.
#> 
#> ──────────────────────────────────────────────────────────
```

