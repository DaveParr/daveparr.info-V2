+++
title = "Why did I make this dev.to API wrapper?"
description = "I like dev.to, I don't like the editor"
date = 2020-06-30
[taxonomies]
tags = ["dev.to", "dev.to.ol", "api"]
[extra]
disclaimer = """
This blog was originally generated from dev.to posts via the Stackbit service. 

Since then StackBit pivoted, was purchased by Netlify, and dev.to discontinued the StackBit integration.
"""
+++

`dev.to.ol` is 0.0.1!

[daveparr/dev.to.ol](https://github.com/daveparr/dev.to.ol)

## What’s in the box?

dev.to.ol has the minimum set of viable functions I think it needs to
operate, and (just) enough testing to keep it stable. The api is
starting to come together and there seems to be at least 1 other person
who cares enough about this project to talk about it to someone else, so
I thought it might be handy to get a little more together about things.
I also wanted to make up a celebration to share a little about why I
made it and what dev.to means to me.

## Are there other similar boxes?

R is in many ways a literate programming language. Markdown is pretty
baked into most of our ecosystem through [R
Markdown](https://rmarkdown.rstudio.com/), as is
[LaTeX](https://en.wikipedia.org/wiki/LaTeX_). I think because of this,
and the frequent usage in academic publishing, R users have developed
tooling for blogging pretty heavily. We have
[blogdown](https://bookdown.org/yihui/blogdown/) and
[distill](https://rstudio.github.io/distill/) which are becoming very
widely used and feature rich, and @maelle has written a pretty
comprehensive guide to all the different approaches in the [Scientific
Blogging with R Markdown
course](https://scientific-rmd-blogging.netlify.app/) including her own
solution for Wordpress,
[goodpress](https://github.com/maelle/goodpress). As Data Scientists
(emphasis on the *science*), we need to share our work reproducibly, and
in a way that encourages understanding and review.

## So why have another box?

I was looking for something to keep me sane while I was furloughed.

Also, I had poked around dev.to about a year before, and was impressed
by what I took to be it’s un-official mission statement of “We aren’t
Medium, we’re what Medium should have been”.

[Medium was never meant to be a part of the developer ecosystem](https://dev.to/devteam/medium-was-never-meant-to-be-a-part-of-the-developer-ecosystem-25a0)

I’m sure there is a lot more to it than that, especially with the recent
[Forem](https://dev.to/devteam/for-empowering-community-2k6h)
announcement, but personally, that was a hook I perceived which really
stuck me.

However, at the time the in built editor was “fine”. Fine enough, but
not great. I’m also used to R Notebooks, where I write my code next to
my prose, and compile the whole thing in one. It seemed valuable to
bring that ability to my dev.to posts. I also was aware of all the
alternatives above, but dev.to offered 2 things that were different to
the above options:

### Hosted and managed service

Don’t get me wrong, I have nothing but love for JAMstack and static
sites in general. I [maintain a template for making them for SatRdays
conferences](https://github.com/satRdays/satRday_site_template).
However, I don’t love the process of doing it. In this case I absolutely
believe in the cause, which is why I continue to volunteer on the
project. However, the process of interacting with static sites doesn’t
actually make me *happy*.

I was looking for the simplest way I could casually blog, which ideally
required no extra work beyond writing articles (to start with).

### Community

I wanted at least a few people to read my articles, and I wanted to read
theirs. I’m sure I could have accomplished this with more work in SEO,
cultivating a social media network and all that jazz, but again: I just
wanted to write some stuff that some people might read, and read their
stuff. dev.to had a discovery feed, it had active users, it had
community. This was just what I was looking for. I also kind of hate
twitter (yes, I still use it when I ‘have’ to).

## Motivation

Initially my work flow sucked. I wrote an `.Rmd` in RStudio. Compiled it
to a GitHub flavoured `.md`. Copy and pasted the output into the editor,
added the meta data, then uploaded all the images. If I found I’d made a
mistake, I’d either do the right thing, which was edit the `.Rmd`,
recompile, re-copy-pasta, or I’d more often do the quick thing which is
enter the browser editor and fix the typo.

## Inception

As I was chewing over the best way to make this work gooder (I am a
programmer after all), a few things happened simultaneously.

### Most of the R users on the site syndicate

There are some really great R users on here already:

[juliasilge](https://dev.to/juliasilge)

[sckott](https://dev.to/sckott)

[maelle](https://dev.to/maelle)

[colinfay](https://dev.to/colinfay)

I found out that most of the R users here I follow are actually
[re-syndicating through rss](https://twitter.com/juliasilge/status/1260580363971317765).

That’s great for them, but I was actually trying to avoid my own site!

### Dev.to has an API

I was poking around older posts and found out that dev.to has an
[API](https://docs.dev.to/api). At the time I thought I might play with
webhooks to boot ‘saved’ articles into pocket to work on my e-reader (at
some point I still might). However, in this case it was kind of perfect.
An API driven work flow between .Rmd in RStudio and the hosted dev.to
community. I could see it in my head, and it’s not like I was busy in
May…

[posting from .Rmd to dev.to](@/blog/posting-from-rmd-to-dev-to-5gld/index.md)
## Evolution

After proving it ‘might’ be doable, I did some work fleshing out the
‘best’ way to do it.

[posting straight from .Rmd to dev.to (for real this time)](@/blog/posting-straight-from-rmd-to-dev-to-1j4p/index.md)

So I started some super-basic minimum viable functions. How do I get the
post to turn up on dev.to? How do I make the meta-data work with the
post? What happens if I’ve published and need to correct a typo? All
solvable, all import features, and all now implemented and tested. I got
to learn a lot about testing API wrappers with `vcr` thanks to @sckott.

[Testing my dev.to.ol package](@/blog/testing-my-dev-to-api-package-with-testthat-webmockr-and-vcr-2dgm/index.md)

The functions in the package changed a bit as I went through user
testing by using it to write my posts on dev.to that you can see here. I
was also able to generate content pretty quick because I could write
about developing the functions that I was testing at the same time by
writing the articles. Virtuous cycles! (/ black holes)

![a robot in a patterned blue suit on a marble floor infinitely crawling
away from a black hole eating
everything](https://media2.giphy.com/media/lKKXOCVviOAXS/giphy.gif?cid=ecf05e471b5c83569df22ec5aad248c62cb864b40d6efed4\u0026rid=giphy.gif)

At this point I was also starting to look for new work, and was
wondering about if I had made the choice backwards. Maybe I did need my
own site after all. Somewhere to put my CV, and that looked a bit more
professional, and maybe didn’t have so much clutter of articles mixed
with my own questions and a bunch of content from other people. Then I
discovered the @stackbit integration!

[You can now generate self-hostable static blogs right from your DEV content via Stackbit](https://dev.to/devteam/you-can-now-generate-self-hostable-static-blogs-right-from-your-dev-content-via-stackbit-7a5)

This meant that I could hand off all the hosting and styling and
management and deploys to them, but still get a hugo repo which I can
tailor how I want, such as removing the `#meta` and `#help` posts, which
wouldn’t be useful to a recruiter, but also automatically put my content
into a presentable site, with a few contextual links to thing like my
GitHub and LinkedIn under my personal [daveparr.info](daveparr.info)
domain.

[I made my dev.to content into a blog to find a new job](@/blog/i-made-my-dev-to-content-into-a-website-to-find-a-new-job-2kn5/index.md)

## Future

The primary future goal is finding a good way to reference images in the
article in a way that dev.to can use. It’s likely that this will end up
being github itself. Additionally better testing is probably on the
cards as I go. The functions and package API have stabilised enough now
that this can be comprehensive, and getting some CI/CD would be nice
too. I’m also planning on working on smarter ways to run analytics on
the data you can get back from the API about your posts. Maybe even an
inbuilt shiny app?

I hope some others of you might find the package useful, and maybe this
might motivate you to share the work you might already be doing in R, or
even pick up R as a new language!

[Photo by Erwan Hesry](https://unsplash.com/photos/WPTHZkA-M4I) via
Unsplash
