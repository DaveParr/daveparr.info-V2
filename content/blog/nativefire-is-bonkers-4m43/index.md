+++
title = "Nativefier is bonkers"
description = "A tool to make a native app out of things that aren't"
date = 2020-06-26
[taxonomies]
tags = ["linux", "electron"]
+++


I made an electron app in 4 lines...

```sh
nativefier "http://musicforprogramming.net/" -n "musicforprogramming"
cd musicforprogramming-linux-x64/
sudo chmod +x musicforprogramming
./musicforprogramming
```

# Motivation

I like music, but don't like music in browser tabs. Basically because i have it open all the time, I want to find and control it easily, and I don't want it cluttering up an area that might be soley focused on work. 

I discovered some neat apps for google play, and wanted to see if there was one for my other go to, [musicforprogramming](http://musicforprogramming.net/). There wasn't, so I just casually googled how to convert a page into an electron app and OMFG!

# Solution

[Nativefier](https://github.com/nativefier/nativefier)

[This post](https://www.todesktop.com/guides/nativefier) and [this one](https://www.addictivetips.com/ubuntu-linux-tips/nativefier-turn-websites-into-linux-apps/) helped iron out some kinks and now I can launch programming music right from my VS code terminal!

![vs code and an electron app of musicforprogramming made with nativefier](https://dev-to-uploads.s3.amazonaws.com/i/7dh4uokd2pt0zxgeye0n.png)

# Extra credit

```sh
alias musicforprogramming="~/Dev/musicforprogramming-linux-x64/musicforprogramming
```

