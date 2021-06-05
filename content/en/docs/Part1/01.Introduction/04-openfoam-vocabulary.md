---
title: "04 - OpenFOAM Vocabulary"
date: 2020-10-20
weight: 8
description: >
  Learn basic terminology used by CFD practitioners ...
---

This page collects some of the most important concepts and definitions related
to CFD and Reservoir Engineering. Make sure you fully understand what most
of these terms refer to before continuing.

-----

CFD
: **C**omputational **F**luid **D**ynamics, I don't have to explain what that is, right?

-----

Discretization
: It's a Math word, refers to the transition from the continuous state
  (of some mathematical concept, eg. an equation, a function) to a discrete
  representation.

-----

Mesh
: Is the discretized version of a continuous domain:
  - A line is _meshed_ into a finite number of segments,
  - A 2D plane can be meshed into a finite number of squares, triangles, 
    and/or any other (preferably) regular 2D geometries
  - And, a volume is meshed into a finite number of cells of any shape
    (hexahedral, tetrahedral, polyhedral ...etc).

-----

Discretization scheme
: The algorithm which handles the discretization of a specific term in an
  equation through a discrete domain.

-----

Pre-Processing
: An important stage in CFD analysis: Describe the physics problem as a valid
  (and perfectly-matching) OpenFOAM model (case)

-----

Simulation
: The process of solving the PDEs (or ODEs) on the meshed domain: Gives nothing
  but a set of text files containing (usually) a huge amount of data.

-----

Post-Processing
: Processing the data resulting from the simulation: May include building new
  data (new calculations) and visualizing some of its features.

-----

BCs (Boundary Conditions)
: The same as in Fluid Dynamics: Constraints applied on physics fields
  (p, U, T, ... etc) at some mesh boundaries.

-----

ICs (Initial Conditions)
: Values of physics fields at internal (non-boundary) cells of the mesh at 
  simulation's starting time.

-----

OpenFOAM Case
: A directory containing all the necessary information to successfully run
  the desired fluid flow simulation.

-----

Function Objects
: What you would call "User Defined Functions" (UDFs) or simply "plugins"in
  commercial software packages.
  For advanced users: C++ objects having function-like behaviour, allowing them
  to run at solver run-time.

-----

OpenFOAM Fork
: When someone makes his own "version" of an open-source software package, it's
  called a fork.
  The word "version" is reserved for the "versions" produced by the original
  owner of the software.
