+++
title = "Tidy Tuesday and space to learn"
description = "Working with non-breaking spaces in R"
date = 2020-05-19
[taxonomies]
tags = ["R", "tidyverse", "tidytuesday", "gotchas"]
[extra]
toc = true
+++

[TidyTusdays](https://github.com/rfordatascience/tidytuesday) are a
weekly R Community event where people learn about RStats by practising
with a different data set each week. Last week the [Cardiff R User
group](https://www.meetup.com/Cardiff-R-User-Group/) worked on the
volcanoes data-set :volcano:, and something in it really tripped me up.
We’ve done this a few times now, and are building up our work in [this
GitHub repo](https://github.com/CaRdiffR/tidy_thursdays).

``` r
library(tidyverse)
volcano <- readr::read_csv('https://raw.githubusercontent.com/rfordatascience/tidytuesday/master/data/2020/2020-05-12/volcano.csv')
```

## Data structure

Let’s have a look at what the data is

``` r
volcano %>% 
  str()
```
{% crt() %}

```
## tibble [958 × 26] (S3: spec_tbl_df/tbl_df/tbl/data.frame)
##  $ volcano_number          : num [1:958] 283001 355096 342080 213004 321040 ...
##  $ volcano_name            : chr [1:958] "Abu" "Acamarachi" "Acatenango" "Acigol-Nevsehir" ...
##  $ primary_volcano_type    : chr [1:958] "Shield(s)" "Stratovolcano" "Stratovolcano(es)" "Caldera" ...
##  $ last_eruption_year      : chr [1:958] "-6850" "Unknown" "1972" "-2080" ...
##  $ country                 : chr [1:958] "Japan" "Chile" "Guatemala" "Turkey" ...
##  $ region                  : chr [1:958] "Japan, Taiwan, Marianas" "South America" "México and Central America" "Mediterranean and Western Asia" ...
##  $ subregion               : chr [1:958] "Honshu" "Northern Chile, Bolivia and Argentina" "Guatemala" "Turkey" ...
##  $ latitude                : num [1:958] 34.5 -23.3 14.5 38.5 46.2 ...
##  $ longitude               : num [1:958] 131.6 -67.6 -90.9 34.6 -121.5 ...
##  $ elevation               : num [1:958] 641 6023 3976 1683 3742 ...
##  $ tectonic_settings       : chr [1:958] "Subduction zone / Continental crust (>25 km)" "Subduction zone / Continental crust (>25 km)" "Subduction zone / Continental crust (>25 km)" "Intraplate / Continental crust (>25 km)" ...
##  $ evidence_category       : chr [1:958] "Eruption Dated" "Evidence Credible" "Eruption Observed" "Eruption Dated" ...
##  $ major_rock_1            : chr [1:958] "Andesite / Basaltic Andesite" "Dacite" "Andesite / Basaltic Andesite" "Rhyolite" ...
##  $ major_rock_2            : chr [1:958] "Basalt / Picro-Basalt" "Andesite / Basaltic Andesite" "Dacite" "Dacite" ...
##  $ major_rock_3            : chr [1:958] "Dacite" " " " " "Basalt / Picro-Basalt" ...
##  $ major_rock_4            : chr [1:958] " " " " " " "Andesite / Basaltic Andesite" ...
##  $ major_rock_5            : chr [1:958] " " " " " " " " ...
##  $ minor_rock_1            : chr [1:958] " " " " "Basalt / Picro-Basalt" " " ...
##  $ minor_rock_2            : chr [1:958] " " " " " " " " ...
##  $ minor_rock_3            : chr [1:958] " " " " " " " " ...
##  $ minor_rock_4            : chr [1:958] " " " " " " " " ...
##  $ minor_rock_5            : chr [1:958] " " " " " " " " ...
##  $ population_within_5_km  : num [1:958] 3597 0 4329 127863 0 ...
##  $ population_within_10_km : num [1:958] 9594 7 60730 127863 70 ...
##  $ population_within_30_km : num [1:958] 117805 294 1042836 218469 4019 ...
##  $ population_within_100_km: num [1:958] 4071152 9092 7634778 2253483 393303 ...
##  - attr(*, "spec")=
##   .. cols(
##   ..   volcano_number = col_double(),
##   ..   volcano_name = col_character(),
##   ..   primary_volcano_type = col_character(),
##   ..   last_eruption_year = col_character(),
##   ..   country = col_character(),
##   ..   region = col_character(),
##   ..   subregion = col_character(),
##   ..   latitude = col_double(),
##   ..   longitude = col_double(),
##   ..   elevation = col_double(),
##   ..   tectonic_settings = col_character(),
##   ..   evidence_category = col_character(),
##   ..   major_rock_1 = col_character(),
##   ..   major_rock_2 = col_character(),
##   ..   major_rock_3 = col_character(),
##   ..   major_rock_4 = col_character(),
##   ..   major_rock_5 = col_character(),
##   ..   minor_rock_1 = col_character(),
##   ..   minor_rock_2 = col_character(),
##   ..   minor_rock_3 = col_character(),
##   ..   minor_rock_4 = col_character(),
##   ..   minor_rock_5 = col_character(),
##   ..   population_within_5_km = col_double(),
##   ..   population_within_10_km = col_double(),
##   ..   population_within_30_km = col_double(),
##   ..   population_within_100_km = col_double()
##   .. )
```
    
{% end %}

## Objective

Lots of character columns, and some with some slightly funky formatting,
such as `/` and variations on a theme with `(s)` and `(es)`. We’ve also
got a bunch of ‘sparse’ data in the columns that start with `major_rock`
or `minor_rock` that look like spaces. R has a rich set of tools for
dealing with missing data a little more effectively, so lets
clean this up by setting the missing data to be explicit `NA`. In this
case, as the column is a character type, we need to `NA_character` to
fill it up.

## Failing solution

``` r
volcano %>%
    mutate_at(
    .vars = vars(starts_with(c("major_rock", "minor_rock"))),
    .funs = ~ case_when(
      . == " " ~ NA_character_,
      TRUE ~ .
    )
  ) %>% 
  select(starts_with(c("major_rock", "minor_rock"))) %>% 
  head() %>% 
  knitr::kable()
```

| major_rock_1               | major_rock_2               | major_rock_3        | major_rock_4               | major_rock_5 | minor_rock_1        | minor_rock_2        | minor_rock_3 | minor_rock_4 | minor_rock_5 |
| :--------------------------- | :--------------------------- | :-------------------- | :--------------------------- | :------------- | :-------------------- | :-------------------- | :------------- | :------------- | :------------- |
| Andesite / Basaltic Andesite | Basalt / Picro-Basalt        | Dacite                |                              |                |                       |                       |                |                |                |
| Dacite                       | Andesite / Basaltic Andesite |                       |                              |                |                       |                       |                |                |                |
| Andesite / Basaltic Andesite | Dacite                       |                       |                              |                | Basalt / Picro-Basalt |                       |                |                |                |
| Rhyolite                     | Dacite                       | Basalt / Picro-Basalt | Andesite / Basaltic Andesite |                |                       |                       |                |                |                |
| Andesite / Basaltic Andesite | Basalt / Picro-Basalt        |                       |                              |                | Dacite                |                       |                |                |                |
| Andesite / Basaltic Andesite |                              |                       |                              |                | Dacite                | Basalt / Picro-Basalt |                |                |                |

Well, that doesn’t quite work. What I want is to have the blank spots
filled up with `NA`. Is it my code? It’s not the most basic solution,
using some of the tidyeval concepts such as `vars` and `funs`. Lets make
it as simple as possible.

``` r
volcano %>% 
  filter(major_rock_5 == " ") %>% 
  knitr::kable()
```

| volcano_number | volcano_name | primary_volcano_type | last_eruption_year | country | region | subregion | latitude | longitude | elevation | tectonic_settings | evidence_category | major_rock_1 | major_rock_2 | major_rock_3 | major_rock_4 | major_rock_5 | minor_rock_1 | minor_rock_2 | minor_rock_3 | minor_rock_4 | minor_rock_5 | population_within_5_km | population_within_10_km | population_within_30_km | population_within_100_km |
| --------------: | :------------ | :--------------------- | :------------------- | :------ | :----- | :-------- | -------: | --------: | --------: | :----------------- | :----------------- | :------------- | :------------- | :------------- | :------------- | :------------- | :------------- | :------------- | :------------- | :------------- | :------------- | ------------------------: | -------------------------: | -------------------------: | --------------------------: |

Odd. I can’t find any values that are just spaces, even though they are
printed out that way! I know they are there, I can see them! Luckily,
the point of these projects is to learn, and to learn from each other in
the group :school:.

## Problem

My buddy [Heather](https://twitter.com/HeathrTurnr) was I think a little
surprised when I demonstrated this, but within a few minutes she’d
worked it out. It’s encoded as a *non_breaking space*. She linked this
blog in our chat about [non-braking
spaces](https://blog.tonytsai.name/blog/2017-12-04-detecting-non-breaking-space-in-r/)
and offered us the cryptic solution:

    "\u00A0"

The blog goes into detail about what this is but the tl;dr is: “It looks
like a space but it’s not and it’s [designed that
way](https://en.wikipedia.org/wiki/Non-breaking_space)”.

So a quick tweak to the code and…

## Functional solution

``` r
volcano %>%
    mutate_at(
    .vars = vars(starts_with(c("major_rock", "minor_rock"))),
    .funs = ~ case_when(
      . == "\u00A0" ~ NA_character_,
      TRUE ~ .
    )
  ) %>% 
  select(starts_with(c("major_rock", "minor_rock"))) %>% 
  head() %>% 
  knitr::kable()
```

| major_rock_1               | major_rock_2               | major_rock_3        | major_rock_4               | major_rock_5 | minor_rock_1        | minor_rock_2        | minor_rock_3 | minor_rock_4 | minor_rock_5 |
| :--------------------------- | :--------------------------- | :-------------------- | :--------------------------- | :------------- | :-------------------- | :-------------------- | :------------- | :------------- | :------------- |
| Andesite / Basaltic Andesite | Basalt / Picro-Basalt        | Dacite                | NA                           | NA             | NA                    | NA                    | NA             | NA             | NA             |
| Dacite                       | Andesite / Basaltic Andesite | NA                    | NA                           | NA             | NA                    | NA                    | NA             | NA             | NA             |
| Andesite / Basaltic Andesite | Dacite                       | NA                    | NA                           | NA             | Basalt / Picro-Basalt | NA                    | NA             | NA             | NA             |
| Rhyolite                     | Dacite                       | Basalt / Picro-Basalt | Andesite / Basaltic Andesite | NA             | NA                    | NA                    | NA             | NA             | NA             |
| Andesite / Basaltic Andesite | Basalt / Picro-Basalt        | NA                    | NA                           | NA             | Dacite                | NA                    | NA             | NA             | NA             |
| Andesite / Basaltic Andesite | NA                           | NA                    | NA                           | NA             | Dacite                | Basalt / Picro-Basalt | NA             | NA             | NA             |

## Full solution

Success! After digging around a little, I discovered `str_trim` and
`str_squish` can be used for this as well to make a perfectly tidy
solution!

``` r
volcano %>%
    mutate_at(
    .vars = vars(starts_with(c("major_rock", "minor_rock"))),
    .funs = ~ case_when(
      str_trim(.) == "" ~ NA_character_,
      TRUE ~ .
    )
  ) %>% 
  select(starts_with(c("major_rock", "minor_rock"))) %>% 
  head() %>% 
  knitr::kable()
```

| major_rock_1               | major_rock_2               | major_rock_3        | major_rock_4               | major_rock_5 | minor_rock_1        | minor_rock_2        | minor_rock_3 | minor_rock_4 | minor_rock_5 |
| :--------------------------- | :--------------------------- | :-------------------- | :--------------------------- | :------------- | :-------------------- | :-------------------- | :------------- | :------------- | :------------- |
| Andesite / Basaltic Andesite | Basalt / Picro-Basalt        | Dacite                | NA                           | NA             | NA                    | NA                    | NA             | NA             | NA             |
| Dacite                       | Andesite / Basaltic Andesite | NA                    | NA                           | NA             | NA                    | NA                    | NA             | NA             | NA             |
| Andesite / Basaltic Andesite | Dacite                       | NA                    | NA                           | NA             | Basalt / Picro-Basalt | NA                    | NA             | NA             | NA             |
| Rhyolite                     | Dacite                       | Basalt / Picro-Basalt | Andesite / Basaltic Andesite | NA             | NA                    | NA                    | NA             | NA             | NA             |
| Andesite / Basaltic Andesite | Basalt / Picro-Basalt        | NA                    | NA                           | NA             | Dacite                | NA                    | NA             | NA             | NA             |
| Andesite / Basaltic Andesite | NA                           | NA                    | NA                           | NA             | Dacite                | Basalt / Picro-Basalt | NA             | NA             | NA             |

Let’s try and breakdown what is happening in this solution by concept,
and then outline the routine in human language to finish.

## Concepts

### Non-breaking spaces

  - the ‘missing’ values are not real spaces, they are *non-breaking
    spaces*.
  - [`stringr::str_trim`](https://stringr.tidyverse.org/reference/str_trim.html)
    and `stringr::str_squish` removes space from either the ends or all
    the way through a character string depending on what else is
    happening in the string and what you need from the solution.
  - [`mutate_at`](https://dplyr.tidyverse.org/reference/mutate_all.html)
    is a buddy of `mutate`, where you use *functional programming* style
    to apply a function over a collection of columns.
  - this means that for the context of evaluation, we will be getting
    `""`, where as previously we were seeing `" "` which was actually
    encoded as `"\u00A0"`

### Variable selection

  - [`starts_with`](https://dplyr.tidyverse.org/reference/select.html#useful-functions)
    is a select helper that is designed for cases when a related value
    is stretch wide across multiple columns with similar names, and
    returns a vector of column names filtered to your criteria.
  - [`vars`](https://dplyr.tidyverse.org/reference/vars.html)
    automatically *quotes* the names of the columns to *evaluate* later
    in context and is almost always used as a wrapper to the `.var =`
    argument when it’s supported by a function.
  - this means that we will be doing on operation on each of the columns
    selected.

### Formula function

  - `.funs =` argument, like `.var =`, has a counterpart
    [`funs()`](https://dplyr.tidyverse.org/reference/funs.html), but
    this is being deprecated in favour of the *expression notation*.
  - [`case_when`](https://dplyr.tidyverse.org/reference/case_when.html)
    is an alternative version of the more common `if else` operation.
  - `~` is a special operator that is key to the expression notation. It
    effectively separates the Left Hand Side (LHS) of an expression from
    the Right Hand Side (RHS). It’s used in two ways in this code.
    `case_when` uses full expressions to represent what should happen on
    the RHS when the criteria of the LHS is met. `.funs =` uses it to
    make a lambda style formula. This is sometimes referred to as a
    [*quosure*](https://dplyr.tidyverse.org/articles/programming.html#quoting).
  - `.` is also a special operator, and I recommend reading the
    [documentation of
    pipe](https://magrittr.tidyverse.org/reference/pipe.html) `%>%`. The
    idea of it here is to reference the data being operated on itself.
    In this specific case, it’s each value from each of the selected
    columns for equivalence to an empty space.

## Step-by-step

Did you follow all that? It’s a minefield I know, but it allows us to do
something very powerful in only a few lines. As an alternative way of
understanding what this does, here’s the step by step:

1)  Get all the columns that start with either `"major_rock'` or
    `"minor_rock"`
2)  For each of the those columns trim any value at the start or end
    that is whitespace, including non-breaking white space temporarily,
    without modifying the underlying data
3)  If the value after that is an empty string, replace it in the same
    column with the `NA` value for characters
4)  If the value after does not pass that test, use the original value

So do you *have to* program this way? No, not really. You could manually
create the list of columns you want to modify, but that would be prone
to human error and what if you end up with `"minor_rock_6"` or
`"major_rock_100"`? You could always make a traditional `if else`
structure, but that can get long fast if there are multiple conditions
to check for. How about using the character string `"\u00A0"` to test
for equivalence? Well, that would work now, but how do you pick up if
they change suddenly to actual spaces? Or another whitespace encoding?
Programming like this keeps code readable, maintainable, and robust.

Is this overkill for tidy Tuesdays? Yes, absolutely. Are the problems
that you solve with this approach purely academic? No, not at all. Plus,
doing all that work in 129 characters is pretty neat. Excluding spaces.