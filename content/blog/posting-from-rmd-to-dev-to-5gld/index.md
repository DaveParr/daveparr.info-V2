+++
title = "Posting from .Rmd to dev.to"
description = "A package to post from Rmarkdown to dev.to"
date = 2020-05-20
[taxonomies]
tags = ["R", "package", "api", "dev.to", "literate programming", "dev.to.ol"]
[extra]
toc = true
+++

I’ve started making an R package called
[`dev.to.ol`](https://github.com/DaveParr/dev.to.ol). Dev.to has an [api
which is in beta](https://docs.dev.to/api/), which I’m using to power
the package.

## Prototype

The first function was really just to make sure I could use the api at
all. It gets all the articles for the authenticated user. What’s my most
recent articles title?

``` r
library(dev.to.ol)
dev.to.ol::get_users_articles()[[1]]$title
```
{% crt() %}
```
## Using DEVTO in .Reinviron

## [1] "Tidy Tuesday and space to learn"
```
{% end %}

So how does this function work?

``` r
#' @title get the authenticated users articles
#' @description Provides lots of info on your users articles
#' @param key the api you have set up on DEV.TO
#' @return article stuff
#' @details if no key is supplied, will check for key named DEVTO in `.Renviron`
#' @examples
#' \dontrun{
#' if(interactive()){
#'  get_users_articles("my_api_key")
#'  }
#' }
#' @seealso
#'  \code{\link[httr]{content}},\code{\link[httr]{GET}},\code{\link[httr]{add_headers}}
#' @rdname get_users_articles
#' @export
#' @importFrom httr content GET add_headers

get_users_articles <- function(key = NA) {
  httr::content(httr::GET(url = "https://dev.to/api/articles/me",
                          httr::add_headers("api-key" =
                                              if (!is.na(key)) {
                                                key
                                              } else {
                                                message("Using DEVTO in .Renviron")
                                                Sys.getenv(x = "DEVTO")
                                              })))
}
```

Very simply


  - It uses `httr` to do most of the work through a `GET` request.
  - It allows a user to supply their own api key as an argument.
  - If the user has left that argument as the default `NA`, it will use
    the environmental variable named `DEVTO`. This can be set with an
    `.Renviron` file at the project or user level that looks like this:

<!-- end list -->

    DEVTO="obviouslynotmyrealkey"

## Motivation

So as R users, we have a great tool baked into our language: rmarkdown.
I use it for nearly everything. We also have some great tools to magnify
it’s power, such as R Notebooks, blogdown, package down and distill.
Some great R people are making the effort to put content here such as
Julia Silge (@juliasilge) and Colin Fay (@colinfay), though they are
already well established bloggers, and are publishing from their main
website through RSS. It’s a great solution for them but personally I was
looking at dev.to as a great way to *not* have to have a whole website.
So what do you do if you want to publish a blog, but don’t want to
actually maintain a full website? What if you also want to be able to be
part of the great dev.to community? What if you’re both of those things
and also find you have a large volume of time on your hands? You write a
package to put `.Rmd`s onto dev.to as simply as possible.

## Workflow

My current process is this:

1.  Write an Rmarkdown
2.  Render to `github_document` style markdown
3.  Open the markdown file I just made
4.  Copy and paste the output to the dev.to UI
5.  Fill in the meta-date
6.  Hit publish

An ideal process would be:

1.  Write an Rmarkdown
2.  Hit publish

Lets see, that’s…

``` r
removed_work <- 4/6

scales::label_percent()(removed_work)
```

{% crt() %}
```
## [1] "67%"
```
{% end %}

Wow, gains


## Minimum Viable Function

So, the key piece of the package is to get a function that will jump my
markdown output from my computer onto dev.to. There are lots more bits
that I should have, but that part is the most important.

``` r
#' @title Post a markdown file to dev.to
#' @description Create a new post well rendered markdown file
#' @param key Your API key, Default: NA
#' @param file The path to the file, Default: file
#' @return The response
#' @details Will look for an api key in the `.REnviron` file. Seems to check if the body is identical to a previous article and error if so with `"Body markdown has already been taken"`.
#' @examples
#' \dontrun{
#' if(interactive()){
#'  post_new_article("./articles/my_article.md")
#'  }
#' }
#' @seealso
#'  \code{\link[httr]{POST}},\code{\link[httr]{add_headers}},\code{\link[httr]{verbose}},\code{\link[httr]{content}}
#'  \code{\link[readr]{read_file}}
#' @rdname post_new_article
#' @export
#' @importFrom httr POST add_headers verbose content
#' @importFrom readr read_file

post_new_article <- function(key = NA, file = file) {

  response <- httr::POST(
    url = "https://dev.to/api/articles",
    httr::add_headers("api-key" =
                        if (!is.na(key)) {
                          key
                        } else {
                          message("Using DEVTO in .Renviron")
                          Sys.getenv(x = "DEVTO")
                        }),
    body = list(article = list(
      title = 'title',
      body_markdown = readr::read_file(file = file)
    )),
    encode = 'json',
    httr::verbose()
  )
  httr::content(response)
}
```

Lots of similar things as the earlier function. Api key is all the same
code (don’t worry, when I have to write it a third time, I’ll abstract
it ;P). There are three key changes.

1.  use `httr::POST` instead of `httr::GET` because here I’m giving the
    api info, not requesting it.
2.  use the `body` argument to enclose the info I am sending the api,
    with a list of 1 item `article` which itself is a list of 2 items
    `title` and `body_markdown`
3.  use `readr::read_file` to read the markdown file I want to post into
    memory so it can be sent to the api

## So does it work?

If you can read this, yes 🦾
