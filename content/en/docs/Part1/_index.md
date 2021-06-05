---
title: "Part 1: OpenFOAM for novices"
linkTitle: "Part 1: OpenFOAM for novices"
date: 2020-01-01
weight: 1
description: >
  For OpenFOAM beginners to step up their game and acquire Advanced-Level skills
  (as OpenFOAM users)
  by learning every step in OpenFOAM's workflow 
---

## What does this part prensent?

This part of the course is optimized for OpenFOAM beginners
(which are usually also Linux-novices)
so that they get comfortable running OpenFOAM cases and processing their
results.

The whole part is an introduction to the course's core
(the second part).
So it may seem "fast-paced" for some learners.

Mainly, you'll get introduced to the software packages used throughout
the course, set up a production labs environment to follow along
and solve the assignments.

Then, this part presents a One-Thousand-Foot view of a general simulation
workflow by comparing manual and OpenFOAM-based solution workflows for a
simplified transport problem. This is what the module
__OpenFOAM Simulation in a nutshell__ is for.

After that, we'll dig a little deeper in some simulation workflow steps:

1. __The meshing step:__ Where we investigate some mesh tools and
   the quality of generated meshes.
2. __Equation discretization:__ Where we discuss finite volume schemes
   (from a practical point of view) and their usage in Reservoir Engineering
   context.
3. __Case preparation and results processing:__ Where we explore more productive
   ways to setup OpenFOAM cases including the useful `functionObjects`, and even
   the powerful SWAK4FOAM package.

At this point, the learner can already consider himself an intermediate-user
(if he has enough experience of course), but there is more!

Towards the end-goal of performing design-optimization studies,
we first introduce PyFOAM, a Python-based package which can incredibly simplify
the process of automating the simulation workflow steps learned up to this point
(If you know some Python of course).

Once we're done with that, we get into __Parameterization and Optimization of
OpenFOAM Cases__
where you'll learn to parametrize OpenFOAM cases using both PyFOAM and DAKOTA.

I would call anyone who has survived this far an __advanced OpenFOAM user__. But for this user to
become an advanced __reservoir-engineering-minded__ OpenFOAM user, he has to go through the last
(but completely optional) module: Porous Media Simulations.

## Why do I want to go through this part?

- __What is it good for?__
  - Becoming an advanced OpenFOAM user.
  - Looking at the Reservoir Engineering side of things is not compulsory. So, even if you have nothing to do with
    this field, you can still benefit from this part.

- __What is it not good for?__
  - There is little-to-no OpenFOAM-coding here, take a look at the second part if 
    that's what interests you.
  - Unless you consider meddling with SWAK4FOAM and PyFOAM "coding".
    Take a look at those modules if unsure.

## Part 1 Objectives and Prerequisites

This part focuses on getting some experience in running OpenFOAM cases to solve
sophisticated physics problems. It is field-agnostic mostly, but all units
will consider common reservoir engineering problems as training material.

Hence, the following requirements:

1. A good understanding of basic __Linear Algebra__ constructs:
  - Knowing some theory behind numerical solvers (NSs from now on).
    - Knowledge of at least the conjugate gradient algorithm is a must
    - More advanced Krylov-Space algorithms (Eg. GMRES) is recommended but can
      be skipped for e time being
    - Basic Knowledge of multi-grid NSs is "nice to have"

2. Linux __Command-line__: The part labs are run on Linux machines, so,
   being familiar with some Shell commands and utilities is nice
  - SSH (Log)into the cloud machines
  - Perform basic file/directory manipulations and potentially some text
    editing.

3. __Programming skills__: You probably can pick up the important things as you
  go; but a pre-experience with the following languages might be helpful
  - C++: Because OpenFOAM is written in C++.
  - Python: That's what we use to ease case pre-and-post processing.

4. __Git skills__: The assignments and course projects will use git to
  version-control OpenFOAM cases. So, you should be faimilar with the following
  git operations
  - Fork a repo on Github (Or similar)
  - Clone a repo to your local machine, make changes, commit and push them
  - Open Pull requests and issues on github
  - Merge branches and resolve conflicts

5. __Reservoir Enginnering__ knowledge is not a hard prerequisite, but it's
   recommended to:
  - Take a look at Buckley-Levrett theory
  - Get familiarized with common CFD equations: mass conservation, Navier
    Stocks equations as well as Darcy's flow in porous mediums

6. "__Can do it__" attitude is greately encouraged. Even if something breaks;
  It's helpful to have the attitude of "Let's dig in and make it work". Even
  if all you can do is break it further; just attempting to fix things is a
  decent gain.

## Potentially Useful tools

- __A text/code editor__: You'll need a decent text (or code) editor to
  effectively edit case files. Preferably, one that can easily edit files on
  remote servers (over SSH connections). I prefer (Neo)VIM because it's
  available on most systems by default, but there are many other options:
  Emacs, Atom and Sublime Text to name a few. Some people also prefer using
  IDEs; you can do that too.
- __Docker__: Running things in "containerized" Linux instances is
  "good-n-common" practice across fields.
  It's more secure and convenient. Whenever you need to play with Docker Containers, head over to
  [Play-With-Docker](https://labs.play-with-docker.com/)

## Where should I go next?

When you finish this part you should proceed to the read deal (Part 2).<br>
For now, let's set up your environment and introduce you to the world of
OpenFOAM.
