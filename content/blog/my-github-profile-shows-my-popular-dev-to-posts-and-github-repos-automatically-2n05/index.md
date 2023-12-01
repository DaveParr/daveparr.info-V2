+++
title = "My GitHub profile shows my popular DEV.to posts and GitHub repos automatically"
date = 2020-08-05
+++

This is my github profile:

[DaveParr/DaveParr](https://github.com/DaveParr/DaveParr)

## Background

I made it because I saw this repo by zhiiyang

[zhiiiyang/zhiiiyang](https://github.com/zhiiiyang/zhiiiyang)

Thanks to @mokkapps for this article which uses the twitter updating
action for discovering it.

[How I built a self updating readme](https://dev.to/mokkapps/how-i-built-a-self-updating-readme-on-my-github-profile-418d)

## Motivation

It took my a little while to decide what to do with my profile. I wanted
something data driven, and I wanted something to show-off my programming
successes. So I decided to show my most popular repos, and also to show
my most popular dev.to posts! I noticed that zhiiiyang made extensive
use of [`r-lib/actions`](https://github.com/r-lib/actions), and I’ve
found that it was really valuable for my project too!

## Method

### Repos

The repos script was where I started. Building from zhiiiyang’s work, I
built a [GitHub
workflow](https://github.com/DaveParr/DaveParr/blob/5c66bd4a2bd970ec7ad85e6de56fedcc75fbf74f/.github/workflows/main.yml)
to call a script I had written.

The [script was quite
simple](https://github.com/DaveParr/DaveParr/blob/5c66bd4a2bd970ec7ad85e6de56fedcc75fbf74f/repos.R).
It gets my repos from the GitHub API through the
[`gh`](https://github.com/r-lib/gh) package, and tidies the return using
`hoist` to grab the important bits. It then filters and pivots the data
into a simple plot.

The most interesting part was where zhiiiyang added the output as a
commit by the action itself. The authentication for the action is
actually allowed by this section:

``` yaml
env:
    GITHUB_PAT: ${{ secrets.GITHUB_TOKEN }}
```

I did stumble a little over the correct path to the plot image. It turns
out that you can use the relative location (`./graph.png`) which will
work if you are viewing the README from *within* the repo, but to make
it work when the README is displayed from my *user* page you have to use
the absolute path
(`https://github.com/daveparr/daveparr/blob/main/graph.png`).

### Posts

The posts was actually pretty easy once I’d got used to how GitHub
Action operate, and also how the GitHub `README` profiles worked.

The key part was actually a feature I recently developed for my own
package.

[DaveParr/dev.to.ol](https://github.com/DaveParr/dev.to.ol)

`dev.to.ol` is a package to help R users manage their dev.to content. In
particular I had recently finished the functions that return data from
the API about your published articles. The key to this is to have the
DEV.TO api key that you want to use set as an encrypted secret in the
repo. once that is set, my package can read it if it’s set as an
environmental variable along with the `GITHUB_PAT`.

``` yaml
env:
    DEVTO: ${{ secrets.DEVTO }}
    GITHUB_PAT: ${{ secrets.GITHUB_TOKEN }}
```

Because my package returns a tidy data.frame like object, it was trivial
to munge it down to just what I wanted to show, and then format it
neatly with `knitr`. I also went all in on the `r-lib/actions` examples,
and now not only generate the new data for both the chart and blogs
during the GitHub action, but also do the full compile from `.Rmd` to
`.md` using the [`setup-pandoc`
action](https://github.com/DaveParr/DaveParr/blob/1f0d043ead21077879e4ba8bb282d66f9a6e1cb3/.github/workflows/main.yml#L18).

## Lessons in automation

I’d been meaning to explore GitHub Actions for a little while, and I
found a few things out that I’m going to be considering in the future as
I develop this an other projects.

### Pricing vs performance on macOS and Linux

It’s free to a point, and then you need to pay. I had a look at the
[pricing
plan](https://docs.github.com/en/github/setting-up-and-managing-billing-and-payments-on-github/about-billing-for-github-actions)
and noticed that your runtime impacts your pricing. Most of the examples
from `r-lib/actions` run from macos-latest, as does zhiiiyang’s project.
In GitHub Actions pricing a minute of macOS runtime is worth *10
minutes* of linux runtime. I ran on macOS for a while too, but
eventually thought that it might be a smart idea for a long running
personal project to convert to a linux run time, though now I’ve done it
I’m debating going back to mac.

The ‘problem’ is that I did not write this process to be fast, or light.
It’s a silly hobby project to over-automate because I can. Therefore, on
Linux I am now actually [compiling the packages on each installation
*AND* installing libcurl for the api
calls](https://github.com/DaveParr/DaveParr/blob/1f0d043ead21077879e4ba8bb282d66f9a6e1cb3/.github/workflows/main.yml#L21-L29).
The cost savings I make from not picking the faster run time in this
case are approximately negated by the increase in actual run time. By
eye, that part of the job runs at about 10 minutes now, where as on mac
it was about 10 times faster as the pre-compiled binaries could just be
downloaded and would run ‘out of the box’. I’ll probably change it back
in a little while after I’ve checked how variable it can be.

Potential solutions could include some form of caching (which I’ve heard
is maybe supported?) or running the action in my own docker image with
the pre-compiled, though TBH that sounds like work, and this is supposed
to be fun :P

### r-lib actions for the Rmd

I really like the idea of compiling the `README.md` for a package from
the `README.Rmd` we often use in R. I’ve often forgotten in my own work
to do that key step before a push, and having a relatively simple
automation backed into where my repos live will likely be something I
use in the future. The best part of this trick is that `r-libs/actions`
does the most irritating part of ‘making Pandoc work’ for me. So I can
just profit!

## Successs!

I really liked hacking this out. I got to put my `dev.to.ol` package to
another practical lesson and learn about GitHub Actions. Feel free to
re-create this on your profiles, either by grabbing bits or by just
lifting the whole thing. One of the reasons I built it the way I did is
so it could be relatively portable between users, and maybe solve a
problem for more than just me. So if you get this deployed on your
profile, or get stuck, I’d love to hear from you!
