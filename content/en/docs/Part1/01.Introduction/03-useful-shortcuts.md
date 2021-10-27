---
title: "03 - Useful Shortcuts"
date: 2020-10-20
weight: 7
description: >
  So you don't get tired while working on the terminal!
---

While working on OpenFOAM cases, you can use some shell aliases to get around 
faster (These are shell commands):

|Shell Alias | Description |
|---|---|
|`run`| Changes the working directory to the $FOAM_RUN` directory, which is supposed to hold your OpenFOAM cases |
|`tut`| Changes the working directory to the $FOAM_TUTORIALS` directory, which contains standard OpenFOAM tutorials for all standard solvers. |
|`src`| Changes the working directory to the $FOAM_SRC` directory, which contains the source code for all OpenFOAM (standard) libraries and solvers. |
|`doc`| Changes the working directory to the directory containing code documentation (and the user guide) for the current OpenFOAM version. |

-----

These are "just" examples: run the `alias` command to figure out the rest ...

-----

> Note that these aliases are only defined if you have OpenFOAM sourced in your
> shell (run `source path/to/installation/folder/etc/bashrc` to source it).<br>
> OpenFOAM is usually installed to `/opt`, but on the remote container, it's
> in `/home/of/OpenFOAM` but you wouldn't need to worry about that now.

