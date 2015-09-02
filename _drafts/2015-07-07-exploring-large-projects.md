---
layout: post
title: "Exploring large projects"
tags: [Chromium]
---

## Step trough with a debugger, even if you normally don't use it

I never really had to use a debugger before I started working on a large code
base - with smaller projects, I could usually upload the code into my brain and
run a (very imprecise) interpreter in my head.

##  Generate dependency and inheritance graphs

You can use doxygen even if the code you are looking at isn't documented at
all - you mainly want the nice graphs it can provide.

## Use code exploration features in your editor

Until recently, I was using Vim and opened files by tab-completing them on the
command line - if you do this, stop immediately. If you are already using an
IDE, you can probably skip this section.

## Write everything down

Either get a notebook with enough of space to draw, or have an extra tab open
in your favorite editor. You don't have to write coherent sentences and always
be right, nobody else is going to read this. Make assumptions. Write it down
when they are wrong. Write down 3 possible hypothesis on how a certain part of the
system works and try to disprove them all. Embrace your confusion and put it on
paper, swearwords and all. It will get better.


## Archeology

If you are lucky and the project uses some sort of version control, I recommend
at least skimming trough the logs. In the worst case, try to at least get the
oldest possible version of it, even if you have to get it from a floppy that you
found in somebody's drawer.

If you are *extremely* lucky and the project uses git and used it for a while, I
highly recommend using:

    $ git log --reverse --stat
