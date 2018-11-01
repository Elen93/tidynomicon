---
title: "Intellectual Debt"
output: md_document
permalink: /debt/
questions:
  - "FIXME"
objectives:
  - "FIXME"
keypoints:
  - "FIXME"
---



We have accumulated some intellectual debt in the previous lessons,
and we should clear some of before we go on to new topics.

## Don't Use `setwd`

Because [reasons][bryan-setwd].

## But...

No.
Use the [here package][here-package].

## Lazy Evaluation

The biggest difference between Python and R is not that the latter starts counting from 1,
but the fact that R uses [lazy evaluation](../glossary/#lazy-evaluation) for function arguments.
When we write this in Python:


```python
def example(first, second):
    print("first argument is", first)
    print("second argument is", second)
    return first + second
example(1 + 2, 1 / 0)
```

```
## ZeroDivisionError: division by zero
## 
## Detailed traceback: 
##   File "<string>", line 1, in <module>
```

then the message `"starting example"` never appears because expressions are evaluated in this order:

1.  `1 + 2`
2.  `1 / 0` - and fail without getting to the first `print` statement inside the function.

When we write the equivalent R, however, the behavior is rather different:


```r
example <- function(first, second) {
  cat("first argument is", first, "\n")
  cat("second argument is", second, "\n")
  first + second
}

example(1 + 2, 1 / 0)
```

```
## first argument is 3 
## second argument is Inf
```

```
## [1] Inf
```

because the order of evaluation is:

1.  Call `example`.
2.  Evaluate `first` because the first `cat` call needs it.
3.  Invoke `cat` the first time.
4.  Evaluate `second` because the second `cat` call needs it.
5.  Invoke `cat` a second time.
6.  Add the values of the two expressions and return.

Here's another example:


```r
green <- function() {
  cat("green\n")
  10
}

blue <- function() {
  cat("blue\n")
  20
}

combined <- function(left, right) {
  cat("combined\n")
  left + right
}

combined(green(), blue())
```

```
## combined
## green
## blue
```

```
## [1] 30
```

This is not wrong:
it just draws on a different tradition in programming
than languages in the C family (which includes Python).

Lazy evaluation powers many of R's most useful features.
For example,
let's create a tibble whose second column's values are twice those of its first:


```r
t <- tibble(a = 1:3, b = 2 * a)
t
```

```
## # A tibble: 3 x 2
##       a     b
##   <int> <dbl>
## 1     1     2
## 2     2     4
## 3     3     6
```

This works because the expression defining the second column is evaluated *after*
the expression defining the first column.
Without lazy evaluation,
we would be trying to create `b` using `a` in our code (where there isn't a variable called `a`)
rather than inside the function (where `a` will just have been created).
This is why we could write things like:


```r
body <- raw %>%
  select(-ISO3, -Countries)
```

in our data cleanup example:
`select` can evaluate `-ISO3` and `-Countries` once it knows what the incoming table looks like.

In order to make lazy evaluation work,
R relies heavily on a structure called an [environment](../glossary/#environment),
which holds a set of name-value pairs.
Whenever R needs the value of a variable,
it looks in the function's environment,
then in its [parent environment](../glossary/#parent-envrironment),
and so on until it reaches the [global environment](../glossary/#global-environment).
This is more or less the same thing that Python and other languages do,
but R programs manipulate enviroments explicitly more often than programs in most other languages.
To learn more about this,
see the discussion in *[Advanced R][advanced-r]*.

## Formulas

One feature of R that doesn't have an exact parallel in Python
is the formula operator `~` (tilde).
Its original (and still most common) purpose is to provide a convenient syntax
for expressing the formulas used in fitting linear regression models.
The basic format of these formulas is `response ~ predictor`,
where `response` and `predictor` depend on the variables in the program.
For example, `Y ~ X` means,
"`Y` is modeled as a function of `X`",
so `lm(Y ~ X)` means "fit a linear model that regresses `Y` on `X`".

What makes `~` work is lazy evaluation:
what actually gets passed to `lm` in the example above is a formula object
that stores the expression representing the left side of the `~`,
the expression representing the right side,
and the environment in which they are to be evaluated.
This means that we can write something like:


```r
fit <- lm(Z ~ X + Y)
```

to mean "fit `Z` to both `X` and `Y`", or:


```r
fit <- lm(Z ~ . - X, data = D)
```

to mean "fit `Z` to all the variables in the data frame `D` *except* the variable `X`."
(Here, we use the shorthand `.` to mean "the data being manipulated".)

But `~` can also be used as a unary operator,
because its true effect is to delay computation.
For example,
we can use it in the function `tribble` to give names to columns
as we create a tibble on the fly:


```r
temp <- tribble(
  ~left, ~right,
  1,     10,
  2,     20
)
temp
```

```
## # A tibble: 2 x 2
##    left right
##   <dbl> <dbl>
## 1     1    10
## 2     2    20
```

Used cautiously and with restraint,
lazy evaluation allows us to accomplish marvels.
Used unwisely---well,
there's no reason for us to dwell on that,
particularly not after what happened to poor Higgins...

## Magic Names

When we put a function in a pipeline using `%>%`,
that operator calls the function with the incoming data as the first argument,
so `data %>% func(arg)` is the same as `func(data, arg)`.
This is fine when we want the incoming data to be the first argument,
but what if we want it to be second?  Or third?

One possibility is to save the result so far in a temporary variable
and then start a second pipe:


```r
data <- tribble(
  ~left, ~right,
  1,     NA,
  2,     20
)
empties <- data %>%
  pmap_lgl(function(...) {
    args <- list(...)
    any(is.na(args))
  })
data %>%
  transmute(id = row_number()) %>%
  filter(empties) %>%
  pull(id)
```

```
## [1] 1
```

This builds a logical vector `empties` with as many entries as `data` has rows,
then filters data according to which of the entries in the vector are `TRUE`.

A better practice is to use the parameter name `.`,
which means "the incoming data".
In some functions (e.g., a two-argument function being used in `map`)
we can use `.x` and `.y`,
and for more arguments,
we can use `..1`, `..2`, and so on:


```r
data %>%
  pmap_lgl(function(...) {
    args <- list(...)
    any(is.na(args))
  }) %>%
  tibble(empty = .) %>%
  mutate(id = row_number()) %>%
  filter(empty) %>%
  pull(id)
```

```
## [1] 1
```

In this model,
we create the logical vector,
then turn it into a tibble with one column called `empty`
(which is what `empty = .` does in `tibble`'s constructor).
After that,
it's easy to add another column with row numbers,
filter,
and pull out the row numbers.
We used this method in [the warm-up exercise in the previous lesson](../projects/#s:warming-up).

And while we're here:
`row_number` doesn't do what its name suggests.
We're better off using `rowid_to_column`:


```r
data %>% rowid_to_column()
```

```
## # A tibble: 2 x 3
##   rowid  left right
##   <int> <dbl> <dbl>
## 1     1     1    NA
## 2     2     2    20
```

## Copy-on-Modify

Another feature of R that can surprise the unwary is [copy-on-modify](../glossary/#copy-on-modify),
which means that if two or more variables refer to the same data
and that data is updated via one variable,
R automatically makes a copy so that the other variable's value doesn't change.
Here's a simple example:


```r
first <- c("red", "green", "blue")
second <- first
cat("before modification, first is", first, "and second is", second, "\n")
```

```
## before modification, first is red green blue and second is red green blue
```

```r
first[[1]] <- "sulphurous"
cat("after modification, first is", first, "and second is", second, "\n")
```

```
## after modification, first is sulphurous green blue and second is red green blue
```

This is true of nested structures as well:


```r
first <- tribble(
  ~left, ~right,
  101,   202,
  303,   404)
second <- first
first$left[[1]] <- 999
cat("after modification\n")
```

```
## after modification
```

```r
first
```

```
## # A tibble: 2 x 2
##    left right
##   <dbl> <dbl>
## 1   999   202
## 2   303   404
```

```r
second
```

```
## # A tibble: 2 x 2
##    left right
##   <dbl> <dbl>
## 1   101   202
## 2   303   404
```

In this case,
the entire `left` column of `first` has been replaced:
tibbles (and data frames) are stored as lists of vectors,
so changing any value in a column triggers construction of a new column vector.

We can watch this happen using the pryr library:


```r
library(pryr)
```

```
## 
## Attaching package: 'pryr'
```

```
## The following objects are masked from 'package:purrr':
## 
##     compose, partial
```

```r
first <- tribble(
  ~left, ~right,
  101,   202,
  303,   404
)
tracemem(first)
```

```
## [1] "<0x7fb8428e85c8>"
```

```r
first$left[[1]] <- 999
```

```
## tracemem[0x7fb8428e85c8 -> 0x7fb8428ee7c8]: eval eval withVisible withCallingHandlers doTryCatch tryCatchOne tryCatchList tryCatch try handle timing_fn evaluate_call <Anonymous> evaluate in_dir block_exec call_block process_group.block process_group withCallingHandlers process_file knit .f map process main 
## tracemem[0x7fb8428ee7c8 -> 0x7fb8428ee748]: eval eval withVisible withCallingHandlers doTryCatch tryCatchOne tryCatchList tryCatch try handle timing_fn evaluate_call <Anonymous> evaluate in_dir block_exec call_block process_group.block process_group withCallingHandlers process_file knit .f map process main 
## tracemem[0x7fb8428ee748 -> 0x7fb8428ee6c8]: $<-.data.frame $<- eval eval withVisible withCallingHandlers doTryCatch tryCatchOne tryCatchList tryCatch try handle timing_fn evaluate_call <Anonymous> evaluate in_dir block_exec call_block process_group.block process_group withCallingHandlers process_file knit .f map process main 
## tracemem[0x7fb8428ee6c8 -> 0x7fb8428ee688]: $<-.data.frame $<- eval eval withVisible withCallingHandlers doTryCatch tryCatchOne tryCatchList tryCatch try handle timing_fn evaluate_call <Anonymous> evaluate in_dir block_exec call_block process_group.block process_group withCallingHandlers process_file knit .f map process main
```

```r
untracemem(first)
```

This rather cryptic output tell us the address of the tibble,
then notifies us of changes to the tibble and its contents.
We can accomplish something a little more readable using `address`:


```r
left <- first$left # alias
cat("left column is initially at", address(left), "\n")
```

```
## left column is initially at 0x7fb8428ee788
```

```r
first$left[[2]] <- 888
cat("after modification, the original column is still at", address(left), "\n")
```

```
## after modification, the original column is still at 0x7fb8428ee788
```

```r
temp <- first$left # another alias
cat("but the first column of the tibble is at", address(temp), "\n")
```

```
## but the first column of the tibble is at 0x7fb8419dc3c8
```

(We need to uses aliases because `address(first$left)` doesn't work:
the argument needs to be a variable name.)

R's copy-on-modify semantics is particularly important when writing functions.
If we modify an argument inside a function,
that modification isn't visible to the caller,
so even functions that appear to modify structures usually don't.
("Usually", because there are exceptions, but we must stray off the path to find them.)

## Conditions

Cautious programmers plan for the unexpected.
In Python,
this is done by [raising](../glossary/#raise-exception) and [catching](../glossary/#catch-exception) [exceptions](../glossary/#exception):


```python
values = [-1, 0, 1]
for i in range(4):
    try:
        reciprocal = 1/values[i]
        print("index {} value {} reciprocal {}".format(i, values[i], reciprocal))
    except ZeroDivisionError:
        print("index {} value {} ZeroDivisionError".format(i, values[i]))
    except Exception as e:
        print("index{} some other Exception: {}".format(i, e))
```

```
## index 0 value -1 reciprocal -1.0
## index 1 value 0 ZeroDivisionError
## index 2 value 1 reciprocal 1.0
## index3 some other Exception: list index out of range
```

Again, R draws on a different tradition.
We say that the operation [signals](../glossary/#signal-condition) a [condition](../glossary/#condition)
that some other piece of code then [handles](../glossary/#signal-handle).
These things are all simpler to do using the rlang library,
so we begin by loading that:


```r
library(rlang)
```

```
## 
## Attaching package: 'rlang'
```

```
## The following object is masked from 'package:pryr':
## 
##     bytes
```

```
## The following objects are masked from 'package:purrr':
## 
##     %@%, %||%, as_function, flatten, flatten_chr, flatten_dbl,
##     flatten_int, flatten_lgl, invoke, list_along, modify, prepend,
##     rep_along, splice
```

The three built-in kinds of conditions are,
in order of increasing severity,
[messages](../glossary/#message), [warnings](../glossary/#warning), and [errors](../glossary/#error).
(There are also interrupts, which are generated by the user pressing Ctrl-C to stop an operation, but we will ignore those.)
We can signal conditions of these three kinds using the functions `message`, `warning`, and `stop`,
each of which takes an error message as a parameter.


```r
message("This is a message.")
```

```
## This is a message.
```

```r
warning("This is a warning.\n")
```

```
## Warning: This is a warning.
```

```r
stop("This is an error.")
```

```
## Error in eval(expr, envir, enclos): This is an error.
```

Note that we have to supply our own line ending for warnings.
Note also that there are only a few situations in which a warning is appropriate:
if something has truly gone wrong,
we should stop,
and if it hasn't,
we should not distract users from more pressing concerns,
like the odd shadows that seem to flicker in the corner of our eye as we examine the artifacts bequeathed to us by our late aunt.

The bluntest of instruments for handling errors is to ignore them.
If a statement is wrapped in `try`,
errors that occur in it are still reported,
but execution continues.
Compare this:


```r
attemptWithoutTry <- function(left, right){
  temp <- left + right
  "result" # returned
}
result <- attemptWithoutTry(1, "two")
```

```
## Error in left + right: non-numeric argument to binary operator
```

```r
cat("result is", result)
```

```
## Error in cat("result is", result): object 'result' not found
```

with this:


```r
attemptUsingTry <- function(left, right){
  temp <- try(left + right)
  "value returned" # returned
}
result <- attemptUsingTry(1, "two")
cat("result is", result)
```

```
## result is value returned
```

If we are *sure* that we wish to incur the risk of silent failure,
we can suppress error messages from `try`:


```r
attemptUsingTryQuietly <- function(left, right){
  temp <- try(left + right, silent = TRUE)
  "result" # returned
}
result <- attemptUsingTryQuietly(1, "two")
cat("result is", result)
```

```
## result is result
```

Do not do this,
for it will,
upon the day,
leave your soul lost and gibbering in an incomprehensible silent hellscape.
Should you wish to handle conditions rather than ignore them,
you may invoke `tryCatch`.
We begin by raising an error explicitly:


```r
tryCatch(
  stop("our message"),
  error = function(cnd) cat("error object is", as.character(cnd))
)
```

```
## error object is Error in doTryCatch(return(expr), name, parentenv, handler): our message
```

(We need to convert the error object `cnd` to character for printing because it is a list of two elements,
the message and the call,
but `cat` only handles character data.)
Let's use this


```r
tryCatch(
  attemptWithoutTry(1, "two"),
  error = function(cnd) cat("error object is", as.character(cnd))
)
```

```
## error object is Error in left + right: non-numeric argument to binary operator
```

We can handle non-fatal errors using `withCallingHandlers`,
and define new types of conditions,
but this is done less often in day-to-day R code than in Python:
see *[Advanced R][advanced-r]* for details.

## A Few Minor Demons

**Flattening:**
`c(c(1, 2), c(3, 4))` produces `c(1, 2, 3, 4)`,
i.e., `c` flattens the vectors it is passed to create a single-level vector.

**Recursive indexing:**
Using `[[` with a vector subsets recursively:
if `thing <- list(a = list(b = list(c = list(d = 1))))`,
then `thing[[c("a", "b", "c", "d")]]` selects the 1.

**Matrix indexing:**
After `a <- matrix(1:9, nrow = 3)`,
`a[3, 3]` is a vector of length 1 containing the value 9 (because scalars in R are actually vectors),
while `a[1,]` is the vector `c(1, 4, 7)` (because we are selecting the first row of the matrix)
and `a[,1]` is the vector `c(1, 2, 3)` (because we are selecting the first column of the matrix).

**Subsetting data frames:**
When we are working with data frames (including tibbles),
subsetting with a single vector selects columns, not rows,
because data frames are stored as lists of columns.
This means that `df[1:2]` selects two columns from `df`.
However, in `df[2:3, 1:2]`, the first index selects rows, while the second selects columns.

**Repeating things:**
The function `rep` repeats things, so `rep("a", 3)` is `c("a", "a", "a")`.
If the second argument is a vector of the same length as the first,
it specifies how many times each item in the first vector is to be repeated:
`rep(c("a", "b"), c(2, 3))` is `c("a", "a", "b", "b", "b")`.

**Naming elements in vectors:**
R allows us to name the elements in vectors:
if we assign `c(one = 1, two = 2, three = 3)` to `names`,
then `names["two"]` is 2.
We can use this to create a lookup table:


```r
values <- c("m", "f", "u", "f", "f", "m", "m")
lookup <- c(m = "Male", f = "Female", u = "Unstated")
lookup[values]
```

```
##          m          f          u          f          f          m 
##     "Male"   "Female" "Unstated"   "Female"   "Female"     "Male" 
##          m 
##     "Male"
```

**The `order` function:**
While we're on the subject of lookup tables,
the function `order` generates indices to pull values into place rather than push them,
i.e.,
`order(x)[i]` is the index in `x` of the element that belongs at location `i`.
For example:


```r
order(c("g", "c", "t", "a"))
```

```
## [1] 4 2 1 3
```
shows that the value at location 4 (the `"a"`) belongs in the first spot of the vector;
it does *not* mean that the value in the first location (the `"g"`) belongs in location 4.

**Name lookup:**
When you use a name in a function call,
R ignores non-function objects when looking for that value.
For example, the call `orange()` in the code below produces 110
because `purple(purple)` is interpreted as
"pass the value of the local variable `purple` into the globally-defined function `purple`":


```r
purple <- function(x) x + 100
orange <- function() {
  purple <- 10
  purple(purple)
}
orange()
```

```
## [1] 110
```

(True story: Fortran uses `(...)` to mean both "call a function" and "index an array".
It also allows functions and arrays in the same scope to have the same names,
so `P(10)` can mean either "call the function `P` with the value 10"
or "get the tenth element of the array `P`",
depending on which compiler you are using.
Ask not how I know this,
or what curses I uttered upon discovering it
after several hours in a dank basement in Edinburgh...)

**Invisible returns:**
If the value returned by a function isn't assigned to something,
R prints it out.
This isn't always what we want (particularly in library functions),
so we can use the function `invisible` to mark a value
so that it won't be printed by default
(but can still be assigned).
This allows us to convert this:


```r
something <- function(value) {
  10 * value
}
something(2)
```

```
## [1] 20
```

to this:


```r
something <- function(value) {
  invisible(10 * value)
}
something(2)
```

The calculation is still done,
but the output is suppressed.

**Assigning out of scope:**
The assignment operator `<<-` means "assign to a variable in the environment above this one".
As the example below shows,
this means that what looks like creation of a new local variable can actually be modification of a global one:


```r
var <- "original value"
demonstrate <- function() {
  var <<- "new value"
}
demonstrate()
var
```

```
## [1] "new value"
```
  
This is most often used with [closures](#g:closures);
see *[Advanced R][advanced-r]* for more detail.

**One of a set of values:**
The function `one_of` is a handy way to specify several values for matching
without complicated Boolean conditionals.
For example,
`gather(data, key = "year", value = "cases", one_of(c("1999", "2000")))`
collects data for the years 1999 and 2000.

**Functions and columns:**
There's a function called `n`.
It's not the same thing as a column called `n`.


```r
data <- tribble(
  ~a, ~n,
  1,  10,
  2,  20
)
data %>% summarize(total = sum(n))
```

```
## Warning: The `printer` argument is soft-deprecated as of rlang 0.3.0.
## This warning is displayed once per session.
```

```
## # A tibble: 1 x 1
##   total
##   <dbl>
## 1    30
```

```r
data %>% summarize(total = sum(n()))
```

```
## # A tibble: 1 x 1
##   total
##   <int>
## 1     2
```

{% include links.md %}
