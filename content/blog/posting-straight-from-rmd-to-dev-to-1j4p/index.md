+++
title = "Posting straight from Rmd to DEV.TO (for real this time)"
description = "Developing a package to post from Rmarkdown to dev.to"
date = 2020-05-24
[taxonomies]
tags = ["r", "rstats", "dev.to", "dev.to.ol", "api", "package"]
[extra]
toc = true
+++

I’ve spent a little time fleshing out my [open source R package](https://github.com/DaveParr/dev.to.ol) to post from `.Rmd`
straight to dev.to.

## Update from V1

The biggest difference from the [first version](https://dev.to/daveparr/posting-from-rmd-to-dev-to-5gld) is
that there is now a single function to move straight from an `.Rmd` on
disk to a post on dev.to. In V1 the user still had to:

1.  Write an `.Rmd`
2.  `knit` to `github_document`
3.  `post_new_article` using my `dev.to.ol` package
4.  De-duplicate the title
5.  (Optionally) Add meta-data

With this new function the workflow is:

1.  Write an `.Rmd`
2.  `post_new_article` using my `dev.to.ol` package

🎉

``` r
#' @title Post a markdown file to dev.to
#' @description Create a new post from an .Rmd.
#' @param file The path to the file
#' @param key Your API key, Default: NA
#' @return The response
#' @details Will look for an api key in the `.REnviron` file. Seems to check if the body is identical to a previous article and error if so with `"Body markdown has already been taken"`.
#' The following YAML arguments are read from the file YAML frontmatter if present:
#' \describe{
#'   \item{title}{A character string}
#'   \item{series}{A character string}
#'   \item{published}{A boolean}
#'   \item{tags}{list of character strings: \code{["tag1", "tag2"]}}
#' }
#'
#' The default table output method renders a very large print code block.
#' The workaround is to use  \code{\link[knitr]{kable}}.
#'
#' @examples
#' \dontrun{
#' if(interactive()){
#'  post_new_article("./articles/my_article.Rmd")
#'  }
#' }
#' @seealso
#'  \code{\link[rmarkdown]{yaml_front_matter}},\code{\link[rmarkdown]{render}}
#'  \code{\link[readr]{read_file}}
#'  \code{\link[stringr]{str_remove}}
#'  \code{\link[glue]{glue}}
#'  \code{\link[httr]{POST}},\code{\link[httr]{add_headers}},\code{\link[httr]{content}}
#' @rdname post_new_article
#' @export
#' @importFrom rmarkdown yaml_front_matter render
#' @importFrom readr read_file
#' @importFrom stringr str_remove
#' @importFrom glue glue
#' @importFrom httr POST add_headers content

post_new_article <-
  function(file,
           key = NA) {
    check_file <- is_postable_Rmd(file)

    if (check_file) {
      file_frontmatter <- rmarkdown::yaml_front_matter(file)

      output_path <- rmarkdown::render('./data/test.Rmd',
                                       output_format = 'github_document',
                                       output_dir = getwd())

      file_string <- readr::read_file(output_path) %\u003e%
        stringr::str_remove(glue::glue("{title}\n================\n\n\n",
                                       title = file_frontmatter$title))

      response <- httr::POST(
        url = "https://dev.to/api/articles",
        httr::add_headers("api-key" = api_key(key = key)),
        body = list(
          article = list(
            title = file_frontmatter$title,
            series = file_frontmatter$series,
            published = file_frontmatter$published,
            tags = file_frontmatter$tags,
            body_markdown = file_string
          )
        ),
        encode = 'json'
      )
      httr::content(response)
    } else {
      message(attr(check_file, "msg"))
    }
  }
```

## Notable changes

### Extract `YAML` frontmatter and render from `.Rmd` with [`rmarkdown`](https://rmarkdown.rstudio.com/)

The function will now accept an `.Rmd` file directly which it renders to
the correct markdown output internally. This was the biggest part
missing from V1, and was easy to do with `rmarkdown::render`. A great
side-effect is that I can also access the `YAML` frontmatter of the
`.Rmd`, which I can use to populate the meta-data such as the series the
post is in, the tags the post is relevant to, and the published status.

### Check file is suitable with
[`assertthat`](https://github.com/hadley/assertthat)

Another benefit is that there is now a concrete object I can check for
correctness. I’ve adopted the `assertthat` package to help me, and most
of the work was [done in this
commit](https://github.com/DaveParr/dev.to.ol/commit/ddccf8ce60adf2bc35ed61d0c0ae581a9189d32d).
The workhorse function is
[`is_postable_Rmd`](https://github.com/DaveParr/dev.to.ol/blob/master/R/is_postable_Rmd.R):

``` r
is_postable_Rmd <- function(file) {
    assertthat::see_if(
    assertthat::is.readable(file),
    assertthat::has_extension(file, "Rmd")
    )
}
```

Using `assertthat::see_if` was really useful. It allowed me to check for
multiple conditions, and if they were both passed, it returns `TRUE`.
However, if they did not pass, it returns `FALSE` *as well as a message
attribute*. This meant that I could very trivially plumb that back into
the main function for user feedback as to what is the problem with the
line `message(attr(check_file, "msg"))`. Additionally, as I add criteria
to the function I know they will all return human, well formatted error
messages. Great work `assertthat`!

### Deduplicate the title with [`glue`](https://glue.tidyverse.org/) and [`stringr`](https://stringr.tidyverse.org/)

One of the issues with the original approach was the call to render will
generate an `.md` with a title *in the file*, but the dev.to api wants
the title *as a separate part of the call*. `stringr` and `glue` to the
rescue! Using `glue` is basically a super powered paste. It does nice
things to insert named string variables into a templated string. I use
this to make the string that is what I want to *remove* from the
`body_markdown` object, and then do the actual removal with
`str_remove`.

## Next steps

Images. The real power of Rmarkdown comes from turning code into
notebooks populated with graphics representing your data science work. I
have [not yet found an api endpoint for
images](https://dev.to/daveparr/is-there-an-api-endpoint-to-upload-images-to-dev-to-2acd).
The POST looks like it will allow a cover image to be included, but on
the face of it I don’t think it will allow images to be inserted at
arbitrary points in the article text. Still, I’ll start there and see
where it goes!
