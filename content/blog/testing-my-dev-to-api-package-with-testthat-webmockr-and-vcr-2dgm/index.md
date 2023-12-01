+++
title = "Testing my dev.to API package with testthat, webmockr and vcr"
date = 2020-06-03
description = "Technical approaches to testing an API wrapper in R"
[taxonomies]
tags = ["r", "testing", "api", "testthat", "webmockr", "vcr", "dev.to", "dev.to.ol"]
+++

I’ve been working on an [open source R package wrapping the dev.to
API](https://github.com/DaveParr/dev.to.ol). After a bit of prototyping
the core feature and then some iterative development to make it a bit
more useful to a user, it’s starting to settle into something that is
worth testing\\*.

## Testing in R

Sometimes people seem a little surprised that this is something R users/
Data Scientists even think about, but R is a programming language, just
like all the others, except different, just like all the others. Testing
in modern tidyverse R is usually accomplished by the
[`testthat`](https://testthat.r-lib.org/) package. If you are more
familiar with testing in other languages, this quote from the Overview
might help:

> testthat draws inspiration from the xUnit family of testing packages, as well as from many of the innovative ruby testing libraries, like rspec, testy, bacon and cucumber.

### A note on `usethis`

In R we have a rich ecosystem of developer tools to help us make
packages. `testthat` is one. Another one is `usethis`. See what they did
there? Anyway, for the purpose of this article, you just need to know
that a package called `usethis` helps us set up things for us to make
development easier. In the future I might write something more about it
¯\_(ツ)_/¯.

## `testthat`

### Setting up `testthat`

Assuming that you have a well formatted R Package structure, you can
easily enable a testing framework and all the boiler plate you might
need with one line:

``` r
usethis::use_testthat()
```

Which will do a few things for you and tell you what it’s doing.
{% crt() %}
``` sh
✔ Setting active project to '/Users/davidparr/Documents/example.testthat'
✔ Adding 'testthat' to Suggests field in DESCRIPTION
✔ Creating 'tests/testthat/'
✔ Writing 'tests/testthat.R'
● Call `use_test()` to initialize a basic test file and open it for editing.
```
{% end %}

From there, we can use `usethis::use_test()`. If you have a file open in
RStudio which is an `*.R` file containing function definitions it will
then make a new test file for you:

{% crt() %}
``` sh
✔ Increasing 'testthat' version to '>= 2.1.0' in DESCRIPTION
✔ Writing 'tests/testthat/test-hello.R'
● Modify 'tests/testthat/test-hello.R'
```
{% end %}

The contents of that test file will be silly boilerplate, so now it’s
time to do some real work.

### Writing a `testthat` test

``` r
test_that("multiplication works", {
  expect_equal(2 * 2, 4)
})
```

This is the boilerplate. If you run it, you might be surprised that
nothing happens! Well, actually a lot of stuff happens, but it doesn’t
really tell you about it by design. If you make a test that isn’t going
to pass however…

``` r
test_that("maths works", {
  expect_equal(2 * 2, 5)
})
```
{% crt() %}
```
## Error: Test failed: 'maths works'
## * <text>:2: 2 * 2 not equal to 5.
## 1/1 mismatches
## [1] 4 - 5 == -1
```
{% end %}
An error! Just what we wanted! so this is the basis of testing with
`testthat`. Write a `test_that()` function, which has a name, and then a
block of expectations to check against.

## `webmockr`

So this works fine for traditional unit tests, where we can give
discrete calculations, or check that a given function gives a specific
output end to end, where we control both the test, but also the function
as a whole. But what if we’re reliant on some ‘external’ process that we
might not control. The dev.to API for instance? I’m not employed by
dev.to (through I am [looking for a new
opportunity](https://dev.to/daveparr/i-made-my-dev-to-content-into-a-website-to-find-a-new-job-2kn5)),
so I don’t get to play around inside the API system, but I do need to
prove that any code I write will behave the way I want it to based on
their API requests and responses. A simple way to prove this is to
*mock* their service (i.e. impersonate it, not tell it it’s silly). This
is where [`webmockr`](https://docs.ropensci.org/webmockr/) comes in.
`webmockr` is an:

> “R library for stubbing and setting expectations on HTTP requests”

Perfect. Let’s write something using both `testthat` and `webmockr`.

### Writing a `webmockr` test

In `webmockr` we make *stubs*. These are fake, minimal objects that are
similar to test fixtures. We know their properties, because we made
them, and we want to make sure that any functions we write interact with
these objects in a predictable way. Another way of looking at them is as
a fake API. They look like an API, with responses and status codes, but
they only exist in our test suite. This means I don’t need to bombard
dev.to with requests any more to make sure I haven’t broken anything.

``` r
webmockr::enable(adapter = "httr")

webmockr::stub_registry_clear()

webmockr::stub_request("get", "https://dev.to/api/users/me") %>%
  webmockr::to_return(body = "success!", status = 200)

my_user <- dev.to.ol::get_my_user()

test_that("stubbed response is 200", {
  expect_is(my_user, "response")
  expect_equal(my_user$status_code, 200)
})
```

This code first of all sets up our test file to understand that requests
will be sent as if from the `httr` package. It then clears the registry,
just to make sure nothing is left in a cache. It then populates the now
empty cache with a new stub. This stub will respond to a `GET` request
to the URL, and will return a simple text body, and a 200 status code.
The function I want to test is then run, which in this environment hits
the *stub*, not the real api. The object that is returned is then
checked by `test_that` to make sure it is a `response` type object, and
that is has a status code that has the value 200.

### Good enough, but is it actually enough?

This proves a few, specific things. That the function returns a
`response` that has a 200 status code if it trys to `GET` from that
specific URL. However, APIs actually return quite a lot of information
by default and maybe I care about more things than a 200 status code.
They can also have quite complex structures, so using this method to
make a fake response could get *very* awkward if I am trying to make a
realistic response. Also, what if the structure of the api changes
underneath us? It is in beta after all. A big change would mean all
those carefully written pipes would have to be rewritten by me every
time. Blergh. Luckily we have a solution for that too!

## `vcr`

[`vcr`](https://docs.ropensci.org/vcr/) does not stand for `Very Cool
Responses`, but I think it should. From it’s own description:

> Record HTTP calls and replay them

It’s an R port of a ruby gem of the same name, this package allows you
to ‘record’ the response of an API to a YAML file ‘cassette’. You can
then ‘play’ the ‘cassette’ back during the test as if the API was being
actually called. If you’re still not sure where the name comes from,
then you might be a little to young to [get the
reference](https://twitter.com/TheEllenShow/status/1116056186606850048).

### Setting up `vcr`

`vcr` has a few things it needs in a project to run, and though it
doesn’t have its *own* entry in `usethis`, it does have it’s own
set-up function in a similar style:

``` r
vcr::use_vcr()
```

{% crt() %}
``` sh
◉ Using package: vcr.example  
◉ assuming fixtures at: tests/fixtures  
✓ Adding vcr to Suggests field in DESCRIPTION  
✓ Creating directory: ./tests/testthat  
◉ Looking for testthat.R file or similar  
✓ tests/testthat.R: added  
✓ Adding vcr config to tests/testthat/helper-vcr.example.R  
✓ Adding example test file tests/testthat/test-vcr_example.R  
✓ .gitattributes: added  
◉ Learn more about `vcr`: https://books.ropensci.org/http-testing
```
{% end %}

From there, we can use the normal `testthat` flow. Here’s an example
using the `POST` to write a new article.

``` r
test_that("post new article", {
  vcr::use_cassette("post_new_article", {
    new_article <- dev.to.ol::post_new_article(file = "./test.Rmd")
  })

  expect_is(new_article, "response")
  expect_equal(new_article$status_code, 201)
})
```

Well, that’s an easy change! The only difference from the first test is
that we have wrapped the function we are testing in a `use_cassette`
block, inside the `test_that` block. Now, the first time this function
is run, you [get
this](https://github.com/DaveParr/dev.to.ol/blob/master/tests/fixtures/post_new_article.yml).
A huge YAML file that describes the response of the actual API. Now,
every time the test is run, that cassette will get loaded as the ‘mock’,
and it’s so much more developed than our stub! We can test against
anything we want in the response, and even better, the response is
totally human readable.

What about changes? What happens if you make a change to the data you
use to test the function that invalidates the `cassette`? What if the
dev.to spec changes? Easy, all you do is delete the test and re-run. The
function will then go and get a new response, and populate the file
again. Your tests then run against the new file. You can even commit
these to version control. Then you can tell exactly when an API change
occurred, and what was different afterwards.

## Thanks Scott

Both the `webmockr` and `vcr` packages are being maintained by @sckott,
who is an active writer here on dev.to. and I think he’s really worth a
follow. He also works on [ROpenSci](https://ropensci.org/), which I
think is also a really cool project. If you are working with R on a
scientific/research project you should be extra interested.

* Yes, I know that TDD exists. IMHO: No, I don’t think it’s a bad thing, but also no, I don’t think it’s always something to use everywhere all the time.
