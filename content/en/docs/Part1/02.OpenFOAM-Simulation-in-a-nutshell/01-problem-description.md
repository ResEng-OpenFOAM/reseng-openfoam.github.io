---
title: "01 - Problem Description"
date: 2020-10-24
weight: 5
description: >
  The mathematical description of a sample transport problem we'll work on
math: true
mermaid: true
---

The first step in OpenFOAM's workflow is actually "Describing the transport
problem with mathematical expressions".

This first unit of the module is a brief summary to the assumptions and
simplifications made to model our sample transport problem.

## Governing equations

We intend to numerically solve a basic __steady-state__ and __incompressible__
transport of a passive scalar $\phi$; which basically accounts for the
dirvergence and diffusion terms (No source terms for simlicity).

The flow can then be modeled with the following equation:

$\nabla \cdot (\rho \mathbf{U} \phi) = \nabla \cdot (K \nabla \phi)$

Where:

- $\rho$ Fluid density
- $\mathbf{U}$ Velocity vector
- $K$ Diffusivity coefficient
- $\nabla$ Gradient operator
- $\nabla \cdot$ Divergence operator

With some additional simplifications:

- Unidirectional flow
- The diffusivity coefficient is homogeneous and constant in time

The equation then becomes:
$$\frac{\partial (\rho \mathbf{U} \phi)}{\partial x}
= K \frac{\partial^2 \phi}{\partial^2 x}$$

## Problem Description

The general worklfow of solving such equations consists of descretizing both the
flow domain and the equation itself in order to obtain a system of algebraic
equations (one per cell):

![Get algebraic equations](/course/part-1/img/02.OpenFOAM-Simulation-in-a-nutshell/02-get-algebraic-eqns.svg)


By way of explanation, we take the continuous domain (the green-ish one) and
mesh into a set of cells. We then supply initial field values at cell centers
$\phi_0, \phi_1, \phi_2, ...$ and define the boundary conditions (specifying
$\phi_b$ for example).

The next step is to transform the continuous flow equation into a set of
algebraic ones; Basically, relating field value at a cell center to boundary
faces and adjacent cells.

> We'll discuss the details of how OpenFOAM usually derives theses algebraic
> systems in upcoming modules


In our illustration case, the flow domain is

- A "line", extending from $x_r = 0$ to $x_l = 0.9\mathrm{m}$
- Split into 9 segments (cells, where `cellSize` is 0.1m)

On both ends of the "line", we apply Dirichlet-type (fixed-value) boundary
conditions on $phi$, where:

- $\phi (x_r) = 1$
- $\phi (x_l) = 0$

## Learn more

It's recommended that **you** go through this advective-diffusive transport
problem both manually and with the help of OpenFOAM.

From here, you can head strait to the next unit.
