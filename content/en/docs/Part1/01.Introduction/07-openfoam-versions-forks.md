---
title: "OpenFOAM Versions and Forks"
date: 2020-10-23
weight: 12
description: >
  Know the software you use ...
---

People are free to make their own "versions" of OpenFOAM libraries and solvers.
This right is protected by the GPLv3 license (if its terms are followed while
doing so).

As a result of this fact, many variants of the original OpenFOAM code have
appeared over the years. Most of them were built in the desire of tailoring
the code to a specific (CFD-related) industry.

A [comprehensive list](https://openfoamwiki.net/index.php/Forks_and_Variants)
of these variants ( called "forks" ) is present in OpenFOAM Wiki.
But let me introduce you to the most common ones:

### What I call Mainline-OpenFOAM

By Mainline-OpenFOAM, I mean the original version maintained by
"The OpenFOAM Foundation", which can be downloaded from
[openfoam.org](https://openfoam.org/). Its versionning is kept simple:
eg. v4.0, v5.0, v5.x, v7.0 ... etc. You can get the code for each of
these versions at [Mainline OpenFOAM Github Repos](https://github.com/OpenFOAM).

This is what we use for this course. In fact, the `openfoam` docker container
in the labs has the most recent development version of this fork (OK, maybe
there are a few missed commits, but the container will certainly stay on the
most recent stable version).

### OpenCFD Releases

There is also a fork called OpenFOAM**+**, which is maintained by OpenCFD.
It can be downloaded from [openfoam.com](https://www.openfoam.com/).
These guys take what the OpenFOAM foundation does and build on on top of it
(hence the + in the name).

They host the development code on [their own](https://develop.openfoam.com)
and release the code periodically in June and December of each year 
(I suppose, hence the versionning v1806, v1812 ... etc) 

### Foam-Extend

- It was derived from an (and now deprecated) old version of OpenFOAM (3.x)
- It is tailored for community developments, completely open to the general
  public.
- Has some nice features that mainline OpenFOAM doens't have, but
  there is a trade-off of other features.

## What's next?

This appears to be the last unit in this introductory module. The real fun
starts in the next module where we'll actually simulate stuff...
