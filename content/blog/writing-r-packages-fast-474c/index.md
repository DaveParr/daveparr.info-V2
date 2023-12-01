+++
title = "Writing R packages, fast"
date = 2020-06-10
description = "How to use RStudio, usethis and sinew to write R packagesthe easy way"
[taxonomies]
tags = ["r", "rstats", "package", "usethis", "roxygen", "sinew"]
+++

R packages are great. R users have a rich ecosystem of extensions to
help us doing various things. We have our own *integrated* package
management system,
[CRAN](https://cran.r-project.org/doc/FAQ/R-FAQ.html#What-is-CRAN_003f),
and we also have [Metacran](https://www.r-pkg.org/) which gives us an
easy way to work out how popular specific packages are used. R also
makes it ‘easy’ for you to write your own packages! Sort of.

# How can I create and R package?

CRAN suggests making an R package [is really
simple](https://cran.r-project.org/doc/FAQ/R-FAQ.html#How-can-I-create-an-R-package_003f).

> 5.5 How can I create an R package? A package consists of a subdirectory containing a file DESCRIPTION and the subdirectories R, data, demo, exec, inst, man, po, src, and tests (some of which can be missing). The package subdirectory may also contain files INDEX, NAMESPACE, configure, cleanup, LICENSE, LICENCE, COPYING and NEWS.
>  See section “Creating R packages” in Writing R Extensions, for details. This manual is included in the R distribution, see What documentation exists for R?, and gives information on package structure, the configure and cleanup mechanisms, and on automated package checking and building.
> R version 1.3.0 has added the function package.skeleton() which will set up directories, save data and code, and create skeleton help files for a set of R functions and datasets.

So I just run the function `package.skeleton()`, and then I just fill in
some R code right? Well, you could. But also, please don’t. Having done
it that way myself the first few times, you *can* do that, but that
doesn’t mean you *should*. Luckily R is a programming language, and
people use programming languages to automate things. So obviously people
have built packages the slow way that will help you build your package
[easier, better, faster](https://www.youtube.com/watch?v=gAjR4_CbPpQ).
We should use those instead!

# Starting a project

You should be using RStudio. It’s an [Open
Source](https://github.com/rstudio/rstudio) IDE for R specifically,
which is [free to download](https://rstudio.com/). The fastest way to
start your project is to open RStudio, and go to `File > New Project >
New Directory > R Package`. Then give it a name, and make sure to create
a git repository with it.

You now have a Hello World package. It contains a function called
`hello` in the `R` directory, with a few comments and notes about short
cuts. You will be able to **Build** the package, and you can run a
**Check**, which will give you a warning, and run the **Test** which
will give you an error.

You’ll also have a few other files and folders. `man` will hold the
manual, or documentation. The `DESCRIPTION` gives you a bunch of
metadata, and the `NAMESPACE` will only have the line
`exportPattern("^[[:alpha:]]+")`. Don’t worry about that for now, we’ll
fix it in a moment. `.Rbuildignore` is a bit like the `.gitignore`. It
will just contain references for R to *not* use when running **Build**.

# Setting up a package and managing package structure with`usethis`

I talked about [`usethis`](https://usethis.r-lib.org/) in my [last
post](https://dev.to/daveparr/testing-my-dev-to-api-package-with-testthat-webmockr-and-vcr-2dgm),
but didn’t do it nearly the justice it deserves. The best place to start
is [Usage](https://usethis.r-lib.org/#usage) and then have a look at the
[Reference](https://usethis.r-lib.org/reference/index.html).

In my projects I tend to use these functions to set up each one:

- `use_pipe()` to allow the package to use pipes from `magrittr`
- `use_testthat()` to set up tests that for the package
- `use_package()` to include a dependency, such as `ggplot2` or
`readr`
- `use_vignette()` to include long form documentation
- `use_readme_rmd()`/`use_readme_md()` to create Readmes
- `use_badge()` to fill in those Readmes with badges

And then to write my package I tend to use these functions to write it:

- `use_r()` to create a new R file to source
- `use_test()` to create a new test for a specific open R file
- `use_data()` to include data sets in my packages

So you can do all these things manually, and maybe the first time you
should. Maybe you should learn that to use a package you need to modify
the `Imports` field in the `DESCRIPTION`, and maybe you need to learn
that to make `testthat` work in your package you need to make a specific
`test` folder, and then include a `testthat.R` file to make it work.
However, I also don’t need to implement my own class every time I want
to make an object that holds a data matrix either, and I also don’t need
to know how the interpreter *actually* works when I run my `hello`
function. There’s a reason why we don’t write in 1 and 0, and it’s the
same reason I don’t see the value in writing my own `NAMESPACE` files
any more. I can do it, but it’s a simple boring task that I’ve done a
bunch of times before, and that at this stage I’m actually likely to get
more wrong than the computer. Talking of which…

# `NAMESPACE`, self-documenting code and self-coding documentation

The `NAMESPACE` helps your package understand what functions it sources
from what other packages, and what functions it allows to be used. [The
R packages book](http://r-pkgs.had.co.nz/namespace.html) has a chapter
on it that goes into detail, but the key thing to really understand is
*how* to use it.

As an example, lets run `usethis::use_pipe()`. This will set-up our
package to allow us to use the `%>%` operator to chain functions. Our
console tells us:

{% crt() %}
``` r
● Run `devtools::document()`
```
{% end %}

We also have a new file `./R/utils-pipe.R`:

``` r
#' Pipe operator
#'
#' See \code{magrittr::\link[magrittr]{\%>\%}} for details.
#'
#' @name %>%
#' @rdname pipe
#' @keywords internal
#' @export
#' @importFrom magrittr %>%
#' @usage lhs \%>\% rhs
NULL
```

So this is a file that’s all comments? Not quite. This is a Roxygen
comment block with `#'`, and it has special powers. It won’t actually be
executed if this file is sourced, but if we *document* our package, the
process will read these comment blocks, and the `@` tags, and populate
out the documentation in `./man`. As we got prompted though, we need to
run the documentation process manually, so lets do that with
`devtools::document()`.

{% crt() %}
``` sh
Updating making.packages documentation
First time using roxygen2. Upgrading automatically...
Writing NAMESPACE
Loading making.packages
Writing NAMESPACE
Writing pipe.Rd
```
{% end %}

Now we have some changes. Our `NAMESPACE` LOOKS LIKE THIS:

``` r
# Generated by roxygen2: do not edit by hand

export("%>%")
importFrom(magrittr,"%>%")
```

and we have a new `./man/pipe.Rd` file:

``` r
% Generated by roxygen2: do not edit by hand
% Please edit documentation in R/utils-pipe.R
\name{\%>\%}
\alias{\%>\%}
\title{Pipe operator}
\usage{
lhs \%>\% rhs
}
\description{
See \code{magrittr::\link[magrittr]{\%>\%}} for details.
}
\keyword{internal}
```

What’s happened? A few things:

1.  Roxygen was associated with our package build to parse and process
    the comments
2.  Roxygen re-wrote the `NAMESPACE`
3.  Roxygen read the Roxygen comments in the `./R/utils-pipe.R`
4.  Roxygen used those special comments to put an import and an export
    line into the `NAMESPACE`
      - The `export` will allow users to use the `%>%` when they load
        our package
      - The `importFrom` will allow us to use the `%>%` in our own code
        in the package itself
5.  Roxygen populated the manual page for the function so we can see it
    in the RStudio help.

Compare the two comment blocks now. What are the differences?

## Roxygen comments

``` r
#' Pipe operator
#'
#' See \code{magrittr::\link[magrittr]{\%>\%}} for details.
#'
#' @name %>%
#' @rdname pipe
#' @keywords internal
#' @export
#' @importFrom magrittr %>%
#' @usage lhs \%>\% rhs
NULL
```

## Roxygenised `.Rd`

``` r
% Generated by roxygen2: do not edit by hand
% Please edit documentation in R/utils-pipe.R
\name{\%>\%}
\alias{\%>\%}
\title{Pipe operator}
\usage{
lhs \%>\% rhs
}
\description{
See \code{magrittr::\link[magrittr]{\%>\%}} for details.
}
\keyword{internal}
```

So the first big one is formatting. The first one looks (mostly) human
readable, while the second one has a lot more cruft. This cruft is
basically LaTeX. The order has been changed around and few of the labels
are a little different, but there is another big change. `@export` and
`@importFrom` aren’t in the documentation! Those were *only* included
to help Roxygen populate the `NAMESPACE` file. So you can see that those
two tags are actually amongst the most important things to include in
documenting other functions. If you check in the `DESCRIPTION` file we
now *also* have a new `Imports` field in the YAML with `magrittr` being
the only entry. Did you notice that there was no *actual* R code in this
function? It actually explicitly contained `NULL`. All of this was
purely generated from the documentation, but now we actually have a new
piece of fundamental functionality in our package. Weird huh?

# The package skeleton’s connected to the comment bone. The comment bone’s connected to the sinew imports.

So now you’re all prepared to write your own package. You know how to
start a new package project, how to set it up with `usethis` and how to
document functions with Roxygen. Hold up though. That thing about not
having to do the basically the same fiddly boring job multiple times in
each package? I really meant it. The cool R kids don’t even write their
own Roxygen any more.

Go back to your `./R/hello.R` file, and ditch those boring vanilla `#`
comments. Now source the function into your global environment so you
can run it interactively. Now `install.packages("sinew")`, and run
`sinew::makeOxygen(hello)`.

``` r
#' @description FUNCTION_DESCRIPTION

#' @return OUTPUT_DESCRIPTION
#' @details DETAILS
#' @examples 
#' \dontrun{
#' if(interactive()){
#'  #EXAMPLE1
#'  }
#' }
#' @rdname hello
#' @export 
```

So [sinew](https://yonicd.github.io/sinew/) has built us a template for
Roxygen, and put the function name in? Cute, but I don’t need that to
write one word for me. OK, how about something a bit more real world.
I’ve been [making a wrapper for the dev.to
API](https://dev.to/daveparr/posting-straight-from-rmd-to-dev-to-1j4p)
(using all these methods). If I go get my `post_new_article` function
and run `sinew::makeOxygen(post_new_article)`:

{% crt() %}
``` r
#' @title FUNCTION_TITLE
#' @description FUNCTION_DESCRIPTION
#' @param file PARAM_DESCRIPTION
#' @param key PARAM_DESCRIPTION, Default: NA
#' @return OUTPUT_DESCRIPTION
#' @details DETAILS
#' @examples 
#' \dontrun{
#' if(interactive()){
#'  #EXAMPLE1
#'  }
#' }
#' @seealso 
#'  \code{\link[rmarkdown]{yaml_front_matter}},\code{\link[rmarkdown]{render}}
#'  \code{\link[readr]{read_file}}
#'  \code{\link[stringr]{str_remove}}
#'  \code{\link[glue]{glue}}
#'  \code{\link[httr]{POST}},\code{\link[httr]{add_headers}}
#' @rdname post_new_article
#' @export 
#' @importFrom rmarkdown yaml_front_matter render
#' @importFrom readr read_file
#' @importFrom stringr str_remove
#' @importFrom glue glue
#' @importFrom httr POST add_headers
```
{% end %}

This is my output. Sinew inspects my function, and then *derives* the
functions I need to import and adds them to the Roxygen comment block.
It also links the functions in the `@seealso` which will show up in the
docs. Remember that these now get *automatically* added to the
`NAMESPACE` and `DESCRIPTIONS` where they need to be, each time I
document my package. It’s also understood what arguments my function
takes, and populated them with the defaults, if present. Now, why would
you bother writing this by hand any more?

# TL;DR

1.  Start your package in the RStudio IDE as a new project
2.  Set up your package with `usethis`
3.  Populate your Roxygen comments with `sinew`
4.  Profit!
