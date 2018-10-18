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

We have accumulated some intellectual debt in the previous four lessons,
and we should clear some of before we go on to new topics.

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

*This is not wrong.*
It is just different---or rather,
it draws on a different tradition in programming
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

```
## Error in UseMethod("select_"): no applicable method for 'select_' applied to an object of class "function"
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

{% include links.md %}
