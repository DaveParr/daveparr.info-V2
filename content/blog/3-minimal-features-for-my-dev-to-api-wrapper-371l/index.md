+++
title = "3 minimal features for my dev.to api wrapper"
description = "Functions to create a new article, set a main image, and collapse spaces in tags"
date = 2020-06-17
[taxonomies]
tags = ["r", "rstats", "dev.to", "dev.to.ol", "api", "rmarkdown", "rvest", "purrr", "stringr", "glue", "unsplash"]
[extra]
toc = true
+++

I’ve made 3 very small features for my [open source R package wrapping
the dev.to API](https://github.com/DaveParr/dev.to.ol).

## `create_new_article`

Now that the main requirements for the file to post are stabilising,
I’ve written a quick and dirty function to make a boilerplate article:

``` r
create_new_article <-
  function(title,
           series = 'series',
           tags = '["tag1", "tag2"]',
           file = "") {
    boilerplate_frontmatter <-
      glue::glue(
        '---\ntitle: "{title}"\noutput: github_document\nseries: "{series}"\ntags: {tags}\n---'
      )

    cat(boilerplate_frontmatter, file = file)
  }
```

This will use the [`glue`](https://glue.tidyverse.org/) package to put
the strings in the function argument into the right place in the
boilerplate YAML front matter. If then uses `cat` to either print that
to screen, or to create a new file with it, if a file path is supplied.

## `main_image`

If there is a `main_image` parameter in the `YAML` front matter that is
a url of an image, that image will be set as the cover image of the
post. I got this one from a photo by [Julian
Dufort](https://unsplash.com/@juliandufort?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)
on
[Unsplash](https://unsplash.com/s/photos/3?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText).
I *believe* that if you put an unsplash URL into this field that goes
**directly to the image** it is within their
[license](https://unsplash.com/license), though if anyone knows
something to the contrary please let me know. It’s the first time I have
used this service, despite hearing about it for years.

## Collapse spaces tags

I recently fooled myself for a good ten minutes into thinking that there
was a problem with my API code yesterday when I kept getting a 422
response to putting a new article up, when in fact it was that I had a
space character in one of my tags. Now the `post_new_article` function
collapses any spaces it encounters in tags. Achieving this was a breeze
with [`purrr`](https://purrr.tidyverse.org/) and
[`stringr`](https://stringr.tidyverse.org//):

``` r
purrr::map(file_frontmatter$tags, stringr::str_remove_all, " ")
```

This little nugget takes the list of tags, and then maps the function
`str_remove_all` across all the spaces. This isn’t at all exposed to the
user, as it’s non-negotiable from the API side anyway :)
