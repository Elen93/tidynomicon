# Cleaning Up Data

Here is a sample of data from `infant_hiv/raw/infant_hiv.csv`,
where `...` shows values elided to make the segment readable:

```
"Early Infant Diagnosis: Percentage of infants born to women living with HIV...",,,,,,,,,,,,,,,,,,,,,,,,,,,,,
,,2009,,,2010,,,2011,,,2012,,,2013,,,2014,,,2015,,,2016,,,2017,,,
ISO3,Countries,Estimate,hi,lo,Estimate,hi,lo,Estimate,hi,lo,Estimate,hi,lo,...
AFG,Afghanistan,-,-,-,-,-,-,-,-,-,-,-,-,-,-,-,-,-,-,-,-,-,-,-,-,-,-,-,
ALB,Albania,-,-,-,-,-,-,-,-,-,-,-,-,-,-,-,-,-,-,-,-,-,-,-,-,-,-,-,
DZA,Algeria,-,-,-,-,-,-,38%,42%,35%,23%,25%,21%,55%,60%,50%,27%,30%,25%,23%,25%,21%,33%,37%,31%,61%,68%,57%,
AGO,Angola,-,-,-,3%,4%,2%,5%,7%,4%,6%,8%,5%,15%,20%,12%,10%,14%,8%,6%,8%,5%,1%,2%,1%,1%,2%,1%,
... many more rows ...
YEM,Yemen,-,-,-,-,-,-,-,-,-,-,-,-,-,-,-,-,-,-,-,-,-,-,-,-,-,-,-,
ZMB,Zambia,59%,70%,53%,27%,32%,24%,70%,84%,63%,74%,88%,67%,64%,76%,57%,91%,>95%,81%,43%,52%,39%,43%,51%,39%,46%,54%,41%,
ZWE,Zimbabwe,-,-,-,12%,15%,10%,23%,28%,20%,38%,47%,33%,57%,70%,49%,54%,67%,47%,59%,73%,51%,71%,88%,62%,65%,81%,57%,
,,,,,,,,,,,,,,,,,,,,,,,,,,,,,
,,2009,,,2010,,,2011,,,2012,,,2013,,,2014,,,2015,,,2016,,,2017,,,
,,Estimate,hi,lo,Estimate,hi,lo,Estimate,hi,lo,Estimate,hi,lo,...
Region,East Asia and the Pacific,25%,30%,22%,35%,42%,29%,30%,37%,26%,32%,38%,27%,28%,34%,24%,26%,31%,22%,31%,37%,27%,30%,35%,25%,28%,33%,24%,
,Eastern and Southern Africa,23%,29%,20%,44%,57%,37%,48%,62%,40%,54%,69%,46%,51%,65%,43%,62%,80%,53%,62%,79%,52%,54%,68%,45%,62%,80%,53%,
,Eastern Europe and Central Asia,-,-,-,-,-,-,-,-,-,-,-,-,-,-,-,-,-,-,-,-,-,-,-,-,-,-,-,
... several more rows ...
,Sub-Saharan Africa,16%,22%,13%,34%,46%,28%,37%,50%,30%,43%,57%,35%,41%,54%,33%,50%,66%,41%,50%,66%,41%,45%,60%,37%,52%,69%,42%,
,Global,17%,23%,13%,33%,45%,27%,36%,49%,29%,41%,55%,34%,40%,53%,32%,48%,64%,39%,49%,64%,40%,44%,59%,36%,51%,67%,41%,
,,,,,,,,,,,,,,,,,,,,,,,,,,,,,
Indicator definition: Percentage of infants born to women living with HIV... ,,,,,,,,,,,,,,,,,,,,,,,,,,,,,
Note: Data are not available if country did not submit data...,,,,,,,,,,,,,,,,,,,,,,,,,,,,,
Data source: Global AIDS Monitoring 2018 and UNAIDS 2018 estimates,,,,,,,,,,,,,,,,,,,,,,,,,,,,,
"For more information on this indicator, please visit the guidance:...",,,,,,,,,,,,,,,,,,,,,,,,,,,,,
"For more information on the data, visit data.unicef.org",,,,,,,,,,,,,,,,,,,,,,,,,,,,,
```

This is a mess---no, more than that, an affront to what little is left of our sanity.
There are comments mixed with data,
values' actual indices have to be synthesized by combining column headings from two rows
(two thirds of which have to be carried forward from previous columns),
and so on.
We want to create the tidy data found in `infant_hiv//tidy/infant_hiv.csv`:

```
country,year,estimate,hi,lo
AFG,2009,NA,NA,NA
AFG,2010,NA,NA,NA
AFG,2011,NA,NA,NA
AFG,2012,NA,NA,NA
...
ZWE,2016,0.71,0.88,0.62
ZWE,2017,0.65,0.81,0.57
```

To bring this data to a state of grace will take some trial and effort,
which we will do in stages.

## Show Me the Numbers

We will begin by reading the data into a tibble:

```r
# tidy-01.R
library(tidyverse)

raw <- read_csv("infant_hiv/raw/infant_hiv.csv")
head(raw)
```
```output
Missing column names filled in: 'X2' [2], 'X3' [3], 'X4' [4], 'X5' [5], 'X6' [6], ...
> head(raw)
# A tibble: 6 x 30
  `Early Infant D… X2    X3    X4    X5    X6    X7    X8    X9    X10   X11
  <chr>            <chr> <chr> <chr> <chr> <chr> <chr> <chr> <chr> <chr> <chr>
1 NA               NA    2009  NA    NA    2010  NA    NA    2011  NA    NA
2 ISO3             Coun… Esti… hi    lo    Esti… hi    lo    Esti… hi    lo
3 AFG              Afgh… -     -     -     -     -     -     -     -     -
4 ALB              Alba… -     -     -     -     -     -     -     -     -
5 DZA              Alge… -     -     -     -     -     -     38%   42%   35%
6 AGO              Ango… -     -     -     3%    4%    2%    5%    7%    4%
# ... with 19 more variables: X12 <chr>, X13 <chr>, X14 <chr>, X15 <chr>,
#   X16 <chr>, X17 <chr>, X18 <chr>, X19 <chr>, X20 <chr>, X21 <chr>,
#   X22 <chr>, X23 <chr>, X24 <chr>, X25 <chr>, X26 <chr>, X27 <chr>,
#   X28 <chr>, X29 <chr>, X30 <chr>
```

All right:
R isn't able to infer column names,
so it uses the entire first comment string as a very long column name
and then makes up names for the other columns.
Looking at the file,
the second row has years (spaced at three-column intervals)
and the column after that has the ISO3 country code,
the country's name,
and then "Estimate", "hi", and "lo" repeated for every year.
We're going to have to combine what's in the second and third rows,
so we're going to have to do some work no matter which we skip or keep.
Since we want the ISO3 code and the country name,
let's skip the first two rows.
(We'll omit `library(tidyverse)` from the top of the code listing from now on.)

```r
# tidy-02.R
raw <- read_csv("infant_hiv/raw/infant_hiv.csv", skip = 2)
head(raw)
```
```output
1: Missing column names filled in: 'X30' [30]
2: Duplicated column names deduplicated: 'Estimate' => 'Estimate_1' [6], 'hi' => 'hi_1' [7],
'lo' => 'lo_1' [8], 'Estimate' => 'Estimate_2' [9], 'hi' => 'hi_2' [10], 'lo' => 'lo_2' [11],...
> head(raw)
# A tibble: 6 x 30
  ISO3  Countries Estimate hi    lo    Estimate_1 hi_1  lo_1  Estimate_2 hi_2
  <chr> <chr>     <chr>    <chr> <chr> <chr>      <chr> <chr> <chr>      <chr>
1 AFG   Afghanis… -        -     -     -          -     -     -          -
2 ALB   Albania   -        -     -     -          -     -     -          -
3 DZA   Algeria   -        -     -     -          -     -     38%        42%
4 AGO   Angola    -        -     -     3%         4%    2%    5%         7%
5 AIA   Anguilla  -        -     -     -          -     -     -          -
6 ATG   Antigua … -        -     -     -          -     -     -          -
# ... with 20 more variables: lo_2 <chr>, Estimate_3 <chr>, hi_3 <chr>,
#   lo_3 <chr>, Estimate_4 <chr>, hi_4 <chr>, lo_4 <chr>, Estimate_5 <chr>,
#   hi_5 <chr>, lo_5 <chr>, Estimate_6 <chr>, hi_6 <chr>, lo_6 <chr>,
#   Estimate_7 <chr>, hi_7 <chr>, lo_7 <chr>, Estimate_8 <chr>, hi_8 <chr>,
#   lo_8 <chr>, X30 <chr>
```

That's a bit of an improvement,
but why are all the columns `character` instead of numbers?
The two reasons are that:

- our CSV file uses `-` (a single dash) to show missing data, and
- all of our numbers end with `%`, which means those values actually *are* character strings.

We'll tackle the first problem first by setting `na = c("-")` in our `read_csv` call:

```r
# tidy-03.R
raw <- read_csv("infant_hiv/raw/infant_hiv.csv", skip = 2, na = c("-"))
head(raw)
```
```output
1: Missing column names filled in: 'X30' [30]
2: Duplicated column names deduplicated: 'Estimate' => 'Estimate_1' [6], 'hi' => 'hi_1' [7],
'lo' => 'lo_1' [8], 'Estimate' => 'Estimate_2' [9], 'hi' => 'hi_2' [10], 'lo' => 'lo_2' [11],...
> head(raw)
# A tibble: 6 x 30
  ISO3  Countries Estimate hi    lo    Estimate_1 hi_1  lo_1  Estimate_2 hi_2
  <chr> <chr>     <chr>    <chr> <chr> <chr>      <chr> <chr> <chr>      <chr>
1 AFG   Afghanis… NA       NA    NA    NA         NA    NA    NA         NA
2 ALB   Albania   NA       NA    NA    NA         NA    NA    NA         NA
3 DZA   Algeria   NA       NA    NA    NA         NA    NA    38%        42%
4 AGO   Angola    NA       NA    NA    3%         4%    2%    5%         7%
5 AIA   Anguilla  NA       NA    NA    NA         NA    NA    NA         NA
6 ATG   Antigua … NA       NA    NA    NA         NA    NA    NA         NA
# ... with 20 more variables: lo_2 <chr>, Estimate_3 <chr>, hi_3 <chr>, ...
```

That's progress.
We now need to strip the percentage signs and convert what's left to numeric values.
To simplify our lives,
let's get the `ISO3` and `Countries` columns out of the way.
We'll save the ISO3 values for later use
(and because it will illustrate a point about data hygiene that we know we're going to want to make,
but which we don't want to reveal just yet).

```r
# tidy-04.R
raw <- read_csv("infant_hiv/raw/infant_hiv.csv", skip = 2, na = c("-"))
countries <- raw$ISO3
body <- raw %>%
  filter(-ISO3, -Countries)
```
```error
Error in filter_impl(.data, quo) :
  Evaluation error: invalid argument to unary operator.
Calls: %>% ... <Anonymous> -> filter -> filter.tbl_df -> filter_impl
Execution halted
```

Whoops.
It takes us a moment to realize that we should have called `select`, not `filter`.
Once we make that change,
we make progress once again:

```r
# tidy-05.R
raw <- read_csv("infant_hiv/raw/infant_hiv.csv", skip = 2, na = c("-"))
countries <- raw$ISO3
body <- raw %>%
  select(-ISO3, -Countries)
head(body)
```
```output
# A tibble: 6 x 28
  Estimate hi    lo    Estimate_1 hi_1  lo_1  Estimate_2 hi_2  lo_2  Estimate_3
  <chr>    <chr> <chr> <chr>      <chr> <chr> <chr>      <chr> <chr> <chr>
1 NA       NA    NA    NA         NA    NA    NA         NA    NA    NA
2 NA       NA    NA    NA         NA    NA    NA         NA    NA    NA
3 NA       NA    NA    NA         NA    NA    38%        42%   35%   23%
4 NA       NA    NA    3%         4%    2%    5%         7%    4%    6%
5 NA       NA    NA    NA         NA    NA    NA         NA    NA    NA
6 NA       NA    NA    NA         NA    NA    NA         NA    NA    NA
...and complaints about extra fabricated variables...
```

But wait.
Weren't there some aggregate lines of data at the end of our input?
What happened to them?

```r
# tidy-06.R
library(tidyverse)

raw <- read_csv("infant_hiv/raw/infant_hiv.csv", skip = 2, na = c("-"))
countries <- raw$ISO3
tail(countries, n = 25)
```
```output
 [1] "YEM"
 [2] "ZMB"
 [3] "ZWE"
 [4] ""
 [5] ""
 [6] ""
 [7] "Region"
 [8] ""
 [9] ""
[10] ""
[11] ""
[12] ""
[13] ""
[14] ""
[15] ""
[16] "Super-region"
[17] ""
[18] ""
[19] ""
[20] ""
[21] "Indicator definition: Percentage of infants born to women living with HIV..."
[22] "Note: Data are not available if country did not submit data to Global AIDS..."
[23] "Data source: Global AIDS Monitoring 2018 and UNAIDS 2018 estimates"
[24] "For more information on this indicator, please visit the guidance: ..."
[25] "For more information on the data, visit data.unicef.org"
```

We sigh heavily.
How are we to trim this?
Since there is only one file,
we can manually count the number of rows we are interested in
(or rather, open the file with an editor or spreadsheet program, scroll down, and check the line number),
and then slice there.
This is a very bad idea if we're planning to use this script on other files---we should
instead look for the first blank line or the entry for Zimbabwe or something like that---but
let's revisit the problem once we have our data in place.

```r
# tidy-07.R
raw <- read_csv("infant_hiv/raw/infant_hiv.csv", skip = 2, na = c("-"))
sliced <- slice(raw, 1:192)
countries <- sliced$ISO3
tail(countries, n = 5)
```
```output
[1] "VEN" "VNM" "YEM" "ZMB" "ZWE"
```

Notice that we're counting rows *not including* the two we're skipping,
which means that the 192 in the call to `slice` above corresponds to row 195 of our original data:
195, not 194, because we're using the first row of unskipped data as headers and yes,
you are in fact making a faint whimpering sound.
We will revisit the problem of slicing data without counting rows manually, we promise.
(And notice also that we are slicing, *then* extracting the column containing the countries.
We did, in a temporary version of this script,
peel off the countries, slice those, and then wonder why our main data table still had unwanted data at the end.
Vigilance, my friends---vigilance shall be our watchword,
and in light of that,
we shall first test our plan for converting our strings to numbers:

```
# tidy-08.R
fixture <- c(NA, "1%", "10%", "100%")
result <- as.numeric(str_replace(fixture, "%", "")) / 100
result
```
```output
[1]   NA 0.01 0.10 1.00
```

And as a further check:

```r
# tidy-08.R
...as above...
is.numeric(result)
```
```output
[1] TRUE
```

The function `is.numeric` is `TRUE` for both `NA` and actual numbers,
so it is in fact doing the right thing here.

So here is our updated conversion script:

```r
# tidy-09.R
raw <- read_csv("infant_hiv/raw/infant_hiv.csv", skip = 2, na = c("-"))
sliced <- slice(raw, 1:192)
countries <- sliced$ISO3
body <- raw %>%
  select(-ISO3, -Countries)
numbers <- as.numeric(str_replace(body, "%", "")) / 100
is.numeric(numbers)
```
```output
Warning messages:
1: In stri_replace_first_regex(string, pattern, fix_replacement(replacement),  :
  argument is not an atomic vector; coercing
2: NAs introduced by coercion
> is.numeric(numbers)
[1] TRUE
```

Oh dear.
It appears that some function `str_replace` is calling is expecting an atomic vector,
not a tibble.
It worked for our test case because that was a character vector,
but a tibble has more structure than that.

The second complaint is that `NA`s were introduced.
That's troubling because we didn't get a complaint when we had actual `NA`s in our data.
However,
`is.numeric` tells us that all of our results are numbers.
Let's take a closer look:

```r
is.tibble(body)
```
```output
[1] TRUE
```
```r
is.tibble(numbers)
```
```output
[1] FALSE
```

Perdition.
After browsing the data,
we realize that some entries are `">95%"`,
i.e.,
there is a greater-than sign as well as a percentage in the text.
We will need to regularize those before we do any conversions.
Before that,
however,
let's see if we can get rid of the percent signs.
The obvious way is is to use `str_replace(body, "%", "")` to get rid of the percent signs,
but that doesn't work:
`str_replace` works on vectors,
but a tibble is a list of vectors.
Instead,
we can use `map` to apply the function `str_replace` to each column in turn to get rid of the percent signs:

```r
# tidy-10.R
raw <- read_csv("infant_hiv/raw/infant_hiv.csv", skip = 2, na = c("-"))
sliced <- slice(raw, 1:192)
countries <- sliced$ISO3
body <- raw %>%
  select(-ISO3, -Countries)
trimmed <- map(body, str_replace, pattern = "%", replacement = "")
head(trimmed)
```
```output
$Estimate
  [1] NA         NA         NA         NA         NA         NA
  [7] NA         NA         NA         NA         "26"       NA
 [13] NA         NA         NA         ">95"      NA         "77"
...
[205] "7"        "72"       "16"       "17"       ""         ""
[211] ""         ""         ""         ""

$hi
  [1] NA    NA    NA    NA    NA    NA    NA    NA    NA    NA    "35"  NA
 [13] NA    NA    NA    ">95" NA    "89"  NA    NA    "10"  NA    NA    "35"
...
```

The problem now is that `map` produces a raw list as output.
The function we want is `map_dfc`,
which maps a function across the columns of a tibble and returns a tibble as a result.
(There is a corresponding function `map_dfr` that maps a function across rows.)

```r
# tidy-11.R
raw <- read_csv("infant_hiv/raw/infant_hiv.csv", skip = 2, na = c("-"))
sliced <- slice(raw, 1:192)
countries <- sliced$ISO3
body <- raw %>%
  select(-ISO3, -Countries)
trimmed <- map_dfr(body, str_replace, pattern = "%", replacement = "")
head(trimmed)
```
```output
# A tibble: 6 x 28
  Estimate hi    lo    Estimate_1 hi_1  lo_1  Estimate_2 hi_2  lo_2  Estimate_3
  <chr>    <chr> <chr> <chr>      <chr> <chr> <chr>      <chr> <chr> <chr>
1 NA       NA    NA    NA         NA    NA    NA         NA    NA    NA
2 NA       NA    NA    NA         NA    NA    NA         NA    NA    NA
3 NA       NA    NA    NA         NA    NA    38         42    35    23
4 NA       NA    NA    3          4     2     5          7     4     6
5 NA       NA    NA    NA         NA    NA    NA         NA    NA    NA
6 NA       NA    NA    NA         NA    NA    NA         NA    NA    NA
```

Now, about those `">95%"` values...
It turns out that `str_replace` uses regular expressions,
not just direct string matches,
so we can get rid of the `>` at the same time as we get rid of the `%`.
We will check by looking at the first `Estimate` column,
which earlier inspection informed us had at least one `">95%"` in it:

```r
# tidy-12.R

raw <- read_csv("infant_hiv/raw/infant_hiv.csv", skip = 2, na = c("-"))
sliced <- slice(raw, 1:192)
countries <- sliced$ISO3
body <- raw %>%
  select(-ISO3, -Countries)
trimmed <- map_dfr(body, str_replace, pattern = ">?(\\d+)%", replacement = "\\1")
trimmed$Estimate
```
```output
  [1] NA         NA         NA         NA         NA         NA
  [7] NA         NA         NA         NA         "26"       NA
 [13] NA         NA         NA         "95"       NA         "77"
 [19] NA         NA         "7"        NA         NA         "25"
...
```

Excellent.
We can now use `map_dfc` to convert the columns to numeric percentages using an on-the-fly function:

```r
# tidy-13.R

raw <- read_csv("infant_hiv/raw/infant_hiv.csv", skip = 2, na = c("-"))
sliced <- slice(raw, 1:192)
countries <- sliced$ISO3
body <- raw %>%
  select(-ISO3, -Countries)
trimmed <- map_dfr(body, str_replace, pattern = ">?(\\d+)%", replacement = "\\1")
percents <- map_dfr(trimmed, function(col) as.numeric(col) / 100)
head(percents)
```
```output
There were 27 warnings (use warnings() to see them)
# A tibble: 6 x 28
  Estimate    hi    lo Estimate_1  hi_1  lo_1 Estimate_2  hi_2  lo_2 Estimate_3
     <dbl> <dbl> <dbl>      <dbl> <dbl> <dbl>      <dbl> <dbl> <dbl>      <dbl>
1       NA    NA    NA      NA    NA    NA         NA    NA    NA         NA
2       NA    NA    NA      NA    NA    NA         NA    NA    NA         NA
3       NA    NA    NA      NA    NA    NA          0.38  0.42  0.35       0.23
4       NA    NA    NA       0.03  0.04  0.02       0.05  0.07  0.04       0.06
5       NA    NA    NA      NA    NA    NA         NA    NA    NA         NA
6       NA    NA    NA      NA    NA    NA         NA    NA    NA         NA
```

27 warnings is rather a lot,
so let's see what `warnings()` right after the `as.numeric` call tells us:

```r
# tidy-14.R

warnings()
```
```output
Warning messages:
1: In .f(.x[[i]], ...) : NAs introduced by coercion
2: In .f(.x[[i]], ...) : NAs introduced by coercion
3: In .f(.x[[i]], ...) : NAs introduced by coercion
4: In .f(.x[[i]], ...) : NAs introduced by coercion
...
```

The first `Estimates` column looks all right,
so let's have a look at the second column:

```r
# tidy-15.R

trimmed$hi
```
```output
  [1] NA   NA   NA   NA   NA   NA   NA   NA   NA   NA   "35" NA   NA   NA   NA
 [16] "95" NA   "89" NA   NA   "10" NA   NA   "35" NA   NA   "5"  NA   "95" NA
 [31] "36" NA   "1"  NA   NA   NA   "6"  NA   "12" NA   "95" NA   NA   "95" NA
 [46] NA   NA   NA   NA   NA   NA   "36" "1"  "4"  NA   NA   NA   NA   "6"  NA
 [61] NA   NA   NA   NA   "77" NA   NA   NA   NA   NA   NA   NA   NA   NA   "74"
 [76] NA   NA   NA   NA   "2"  NA   NA   NA   NA   NA   NA   NA   "95" NA   NA
 [91] NA   NA   NA   NA   NA   "53" "7"  NA   NA   NA   NA   NA   "44" NA   "9"
[106] NA   NA   NA   NA   NA   NA   NA   NA   "2"  NA   NA   NA   NA   "2"  NA
[121] NA   "69" NA   "7"  NA   NA   NA   "1"  NA   NA   NA   NA   NA   NA   "1"
[136] NA   NA   NA   "95" NA   NA   "75" NA   NA   NA   NA   NA   NA   "13" NA
[151] NA   NA   NA   NA   NA   NA   NA   NA   NA   NA   NA   NA   "11" NA   NA
[166] NA   NA   "1"  NA   NA   NA   "12" NA   NA   NA   NA   NA   NA   "9"  "95"
[181] NA   NA   "16" NA   NA   "1"  NA   NA   NA   NA   "70" NA   ""   ""   "hi"
[196] "30" "29" NA   "32" "2"  NA   "2"  "12" NA   "9"  "89" "22" "23" ""   ""
[211] ""   ""   ""   ""
```

Empty strings.
Why'd it have to be empty strings?
More importantly,
where are they coming from?
Let's backtrack by displaying the `hi` column of each of our intermediate variables...

...and there it is.
We are creating a variable called `sliced` that has only the rows we care about,
but then using the full table in `raw` to create `body`.
Here's our revised script,
in which we check *both* the head and the tail:

```r
# tidy-17.R
raw <- read_csv("infant_hiv/raw/infant_hiv.csv", skip = 2, na = c("-"))
sliced <- slice(raw, 1:192)
countries <- sliced$ISO3
body <- sliced %>%
  select(-ISO3, -Countries)
trimmed <- map_dfr(body, str_replace, pattern = ">?(\\d+)%", replacement = "\\1")
percents <- map_dfr(trimmed, function(col) as.numeric(col) / 100)
head(percents)
```
```output
# A tibble: 6 x 28
  Estimate    hi    lo Estimate_1  hi_1  lo_1 Estimate_2  hi_2  lo_2 Estimate_3
     <dbl> <dbl> <dbl>      <dbl> <dbl> <dbl>      <dbl> <dbl> <dbl>      <dbl>
1       NA    NA    NA      NA    NA    NA         NA    NA    NA         NA
2       NA    NA    NA      NA    NA    NA         NA    NA    NA         NA
3       NA    NA    NA      NA    NA    NA          0.38  0.42  0.35       0.23
4       NA    NA    NA       0.03  0.04  0.02       0.05  0.07  0.04       0.06
5       NA    NA    NA      NA    NA    NA         NA    NA    NA         NA
6       NA    NA    NA      NA    NA    NA         NA    NA    NA         NA
```
```r
tail(percents)
```
```output
# A tibble: 6 x 28
  Estimate    hi    lo Estimate_1  hi_1  lo_1 Estimate_2  hi_2  lo_2 Estimate_3
     <dbl> <dbl> <dbl>      <dbl> <dbl> <dbl>      <dbl> <dbl> <dbl>      <dbl>
1    NA     NA   NA         NA    NA    NA         NA    NA    NA         NA
2    NA     NA   NA         NA    NA    NA         NA    NA    NA         NA
3    NA     NA   NA         NA    NA    NA          0.31  0.37  0.26       0.3
4    NA     NA   NA         NA    NA    NA         NA    NA    NA         NA
5     0.59   0.7  0.53       0.27  0.32  0.24       0.7   0.84  0.63       0.74
6    NA     NA   NA          0.12  0.15  0.1        0.23  0.28  0.2        0.38
```

Comparing this to the raw data file convinces us that yes,
we are now converting the numeric data properly,
which means we are halfway home.

## Cut, Pad, and Stitch

We now have numeric values in `percents` and corresponding ISO3 codes in `countries`.
What we do *not* have is tidy data:
countries are not associated with records,
years are not recorded at all,
and the column headers for `percents` have mostly been manufactured for us by R.
We must now sew these parts together like Dr. Frankenstein's trusty assistant Igor
(who, like all in his trade, did most of the actual work but was given only crumbs of credit).

Our starting point is this:

1. Each row in `percents` corresponds positionally to an ISO3 code in `countries`.
2. Each group of three consecutive columns in `percents` has the estimate, high, and low values
   for a single year.
3. The years themselves are not stored in `percents`,
   but we know from inspection that they start at 2009 and run without interruption to 2017.

Our strategy is to make a list of temporary tables:

1. Take three columns at a time from `percents` to create a temporary tibble.
2. Join `countries` to it.
3. Create a column holding the year in each row and join that as well.

and then join those temporary tables row-wise to create our final tidy table.
We might,
through clever use of `scatter` and `gather`,
be able to do this without a loop,
but at this point on our journey,
a loop is probably simpler.
Here is the addition to our script:

```r
# tidy-18.R

first_year <- 2009
last_year <- 2017
num_years <- (last_year - first_year) + 1
chunks <- vector("list", num_years)
for (year in 1:num_years) {
  end <- year + 2
  temp <- select(percents, year:end)
  names(temp) <- c("estimate", "hi", "lo")
  temp$country <- countries
  temp$year <- rep((first_year + year) - 1, num_rows)
  temp <- select(temp, country, year, everything())
  chunks[[year]] <- temp
}
```

We start by giving names to our years;
if or when we decide to use this script for other data files,
we should extract the years from the data itself.
We then use `vector` to create the storage we are going to need to hold our temporary tables.
We could grow the list one item at a time,
but allocating storage in advance is more efficient
and serves as a check on our logic:
if our loop doesn't run for the right number of iterations,
we will either overflow our list or have empty entries,
either of which should draw our attention.

Within the loop we figure out the bounds on the next three-column stripe,
select that,
and then give those three columns meaningful names.
This ensures that when we join all the sub-tables together,
the columns of the result will also be sensibly named.
Attaching the ISO3 country codes is as easy as assigning to `temp$country`,
and replicating the year for each row is easily done using the `rep` function.
We then reorder the columns to put country and year first
(the call to `everything` inside `select` selects all columns that aren't specifically selected),
and then we assign the temporary table to the appropriate slot in `chunks` using `[[..]]`.

Now comes the payoff for all that hard work:

```r
# tidy-18.R

tidy <- bind_rows(chunks)
tidy <- arrange(tidy, country, year)
```

As its name suggests,
`bind_rows` takes a list of tables and concatenates their rows in order.
Since we have taken care to give all of those tables the same column names,
no subsequent renaming is necessary.
We do,
however,
use `arrange` to order entries by country and year.
Let's have a look at the result:

```r
tidy
```
```output
# A tibble: 1,728 x 5
   country  year estimate    hi    lo
   <chr>   <dbl>    <dbl> <dbl> <dbl>
 1 ""       2009       NA    NA    NA
 2 ""       2010       NA    NA    NA
 3 ""       2011       NA    NA    NA
 4 ""       2012       NA    NA    NA
 5 ""       2013       NA    NA    NA
 6 ""       2014       NA    NA    NA
 7 ""       2015       NA    NA    NA
 8 ""       2016       NA    NA    NA
 9 ""       2017       NA    NA    NA
10 AFG      2009       NA    NA    NA
```

What fresh hell is this?
Why do some rows have empty strings where country codes should be
and `NA`s for the three percentages?
Is our indexing off?
Have we somehow created one extra row for each year with nonsense values?

No.
It is not our tools that have failed us, or our reason, but our data.
("These parts are not fresh, Igor---I must have *fresh* parts to work with!")
Let us do this:

```r
# tidy-19.R
raw <- read_csv("infant_hiv/raw/infant_hiv.csv", skip = 2, na = c("-"))
missing <- raw %>%
  filter(is.na(Countries) | (Countries == "") | is.na(ISO3) | (ISO3 == "")) %>%
  select(Countries, ISO3)
missing
```
```output
# A tibble: 21 x 2
   Countries                       ISO3 
   <chr>                           <chr>
 1 Kosovo                          ""   
 2 ""                              ""   
 3 ""                              ""   
 4 ""                              ""   
 5 Eastern and Southern Africa     ""   
 6 Eastern Europe and Central Asia ""   
 7 Latin America and the Caribbean ""   
 8 Middle East and North Africa    ""   
 9 North America                   ""   
10 South Asia                      "" 
# ... with 11 more rows
```

The lack of ISO3 country code for the region names doesn't bother us,
but Kosovo is definitely a problem.
[According to Wikipedia](https://en.wikipedia.org/wiki/ISO_3166-1_alpha-3),
UNK is used for Kosovo residents whose travel documents were issued by the United Nations,
so we will fill that in with an ugly hack immediately after loading the data:

```r
# tidy-20.R
raw <- read_csv("infant_hiv/raw/infant_hiv.csv", skip = 2, na = c("-"))
raw$ISO3[raw$Countries == "Kosovo"] <- "UNK"
missing <- raw %>%
  filter(is.na(Countries) | (Countries == "") | is.na(ISO3) | (ISO3 == "")) %>%
  select(Countries, ISO3)
missing
```
```output
# A tibble: 20 x 2
   Countries                ISO3                                               
   <chr>                    <chr>                                              
 1 ""                       ""                                                 
 2 ""                       ""                                                 
 3 ""                       ""                                                 
 4 Eastern and Southern Af… ""                                                 
 5 Eastern Europe and Cent… ""                                                 
 6 Latin America and the C… ""                                                 
 7 Middle East and North A… ""                                                 
 8 North America            ""    
```

All right.
Let's add that hack to our script,
then save the result to a file.
The whole thing is now 38 lines long:

```r
# tidy-21.R
library(tidyverse)

# Constants.
raw_filename <- "infant_hiv/raw/infant_hiv.csv"
tidy_filename <- "infant_hiv/tidy/infant_hiv.csv"
num_rows <- 192

# Get and clean percentages.
raw <- read_csv(raw_filename, skip = 2, na = c("-"))
raw$ISO3[raw$Countries == "Kosovo"] <- "UNK"
sliced <- slice(raw, 1:num_rows)
countries <- sliced$ISO3
body <- sliced %>%
  select(-ISO3, -Countries)
trimmed <- map_dfr(body, str_replace, pattern = ">?(\\d+)%", replacement = "\\1")
percents <- map_dfr(trimmed, function(col) as.numeric(col) / 100)

# Separate three-column chunks and add countries and years.
first_year <- 2009
last_year <- 2017
num_years <- (last_year - first_year) + 1
chunks <- vector("list", num_years)
for (year in 1:num_years) {
  end <- year + 2
  temp <- select(percents, year:end)
  names(temp) <- c("estimate", "hi", "lo")
  temp$country <- countries
  temp$year <- rep((first_year + year) - 1, num_rows)
  temp <- select(temp, country, year, everything())
  chunks[[year]] <- temp
}

# Combine chunks and order by country and year.
tidy <- bind_rows(chunks)
tidy <- arrange(tidy, country, year)

# Save.
write_csv(tidy, tidy_filename)
```

It works ("It's alive!"),
but we can do better.
Let's start by using a pipeline for the code that extracts and formats the percentages:

```r
# Constants...

# Get and clean percentages.
raw <- read_csv(raw_filename, skip = 2, na = c("-"))
raw$ISO3[raw$Countries == "Kosovo"] <- "UNK"
sliced <- slice(raw, 1:num_rows)
countries <- sliced$ISO3
percents <- sliced %>%
  select(-ISO3, -Countries) %>%
  map_dfr(str_replace, pattern = ">?(\\d+)%", replacement = "\\1") %>%
  map_dfr(function(col) as.numeric(col) / 100)

# Separate three-column chunks and add countries and years...

# Combine chunks and order by country and year...

# Check.
write_csv(tidy, "temp.csv")
```

The two changes are:

1. We use a `%>%` pipe for the various transformations involved in creating percentages.
2. We write the result to `temp.csv` so that we can compare it to the file created by our previous script.
   We should always do this sort of comparison when refactoring code in ways that isn't meant to change output;
   if the file is small enough to store in version control,
   we could overwrite it and use `git diff` or something similar to check whether it has changed.
   However,
   we would then have to trust ourselves to be careful enough not to accidentally commit changes,
   and frankly,
   we are no longer sure how trustworthy we are...

After checking that this has not changed the output,
we pipeline the computation in the loop:

```
# tidy-23.R
# Constants...

# Get and clean percentages...

# Separate three-column chunks and add countries and years.
num_years <- (last_year - first_year) + 1
chunks <- vector("list", num_years)
for (year in 1:num_years) {
  chunks[[year]] <- select(percents, year:(year + 2)) %>%
    rename(estimate = 1, hi = 2, lo = 3) %>%
    mutate(country = countries,
           year = rep((first_year + year) - 1, num_rows)) %>%
    select(country, year, everything())
}

# Combine chunks and order by country and year.
tidy <- bind_rows(chunks) %>%
  arrange(country, year)
```

We have introduced a call to `rename` here to give the columns of each sub-table the right names,
and used `mutate` instead of assigning to named columns one by one.
The lack of intermediate variables may make the code harder to debug using print statements,
but certainly makes this incantation easier to read aloud.
