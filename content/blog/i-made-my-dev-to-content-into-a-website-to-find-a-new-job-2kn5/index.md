+++
title = "I made my dev.to content into a website to find a new job"
description = "Rapidly make a blog using stackbit and dev.to"
date = 2020-05-25
[taxonomies]
tags = ["dev.to", "stackbit", "jamstack", "hugo", "netlify", "static site generator"]
[extra]
disclaimer = """
This blog was originally generated from dev.to posts via the Stackbit service. 

Since then StackBit pivoted, was purchased by Netlify, and dev.to discontinued the StackBit integration.
"""
+++


After discovering [this post](https://dev.to/stackbit/project-benatar-publishing-dev-powered-websites-with-stackbit-lfo) and [this post](https://dev.to/devteam/you-can-now-generate-self-hostable-static-blogs-right-from-your-dev-content-via-stackbit-7a5), I decided to use this tooling to get a very simple job done to help me look for work.

## Context

I've been using dev.to a little over the last year, and a lot more over the last month. It makes sense for me to use this content in my job hunt. I also had a languishing personal page. Originally made as a little experiment with hugo, it hadn't seen love in a while, and due to some jankey usage of high quality photos performance wise it was pretty bad. I'd been meaning to overhaul it for a while, and when the dev.to x stackbit collab came to my attention, it was a perfect fit.

## Goals

1. Use the content I've written, and will be continuing to write, to prove I know at least a few things about programming.
1. Get that content wrapped up neatly with a contact page, a statement that I'm looking for work, and a brief about me
1. Use my daveparr.info domain
1. Make the website more performant than it's current iteration
1. Use a clean, simple, slightly professional theme

## Method

1. Read the instructions in @ben 's [post](https://dev.to/devteam/you-can-now-generate-self-hostable-static-blogs-right-from-your-dev-content-via-stackbit-7a5)
1. Go through the creation flow
1. Assign the domain in netlify (I had a netlify hosted JAMstack previously, and my domain was already loaded in there. YMMV)
1. Clone the project @stackbit created from GitHub, and follow the [readme](https://github.com/DaveParr/daveparrinfo/blob/master/README.md) they left there to help you authenticate correctly
1. Update the [boilerplate stuff](https://github.com/DaveParr/daveparrinfo/commit/d8836e723d49ccd13f5655e7f58346dfa4ade17f)
1. Push the changes back to GitHub
1. Profit

## Hiccup

I was a little surprised when my local copy suddenly got all my blog files [that I then just commited](https://github.com/DaveParr/daveparrinfo/commit/1987c701187b45a85d8b0448a8c577c47ab8b2ac), but that seemed to be ok. Pretty sure I've not broken anything :)

## The End

Goals achieved, faster website, with better styling, that will be kept more up to date.

## P.S. 

If you are reading this _on_ my [website](daveparr.info), this is all actually managed through a headless CMS called [dev.to](dev.to). You can use the link to my profile on the left of this article. If you are reading this on dev.to you can see my running website at [daveparr.info](daveparr.info).