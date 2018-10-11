---
title: "Language Speed Flame-wars"
date: 2018-06-XXT08:23:19+10:00
draft: true
tags:
  - AWS
  - Go
  - Python
  - Node
  - Optimization
---

Nothing irritates me more than reading some unsubstantiated post about how 'slow' a given programming langage is.  Actually, *hearing* about it is even worse because people somehow think that because they're speaking the words they don't have to substantiate it.

In my expereince, generally the authors of these posts are written by people with absolutely no expertise in the langauge they are calling 'slow' and offer fraile speed comparisions with code that any resonably competant engineer would be embarassed to be associated with.

Further to that, I have generally found that true experts in a given software language are able to acheive speeds in so-called 'slow languages' that surpass mid-level engineers working on the 'fast language'.

To this, I have started a speed test project.

## How it works

Contributors attempt to use any language supported by AWS lambda.  The use of AWS lambda is for two reasons:

- A) because the underlying infrastructure can be standardised and
- B) because generally I'm interested also in AWS lambda performance

The reason for point B) is that I use this service a lot and would generally like to be more aware of its strengths and weaknesses

### Code submissions

All projects that acheive the same task are nested under a folder.  For example, `fibonacci/` holds all functions that calculate fibonacci sequences.  All functions within that directory are subjected to the same tests.

Any submission to an existing comparision will get deployed and speed tested.  Reports are in Travis.  The lambdas execute under a role that only allows access to lambda and logs.  They do not have external internet conectivity.

Reports are created as a results of standardised tests and report time and memory consumption.  The lambdas run with short timeout periods on multi (??how many) core machines.
