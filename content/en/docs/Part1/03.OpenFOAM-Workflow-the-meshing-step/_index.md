---
title: "1.03 OpenFOAM Workflow: The meshing step"
linkTitle: "1.03 OpenFOAM Workflow: The meshing step"
date: 2020-11-15
weight: 6
description: >
  A deeper look on how the meshing process should be carried out.
---

{{% alert title="Note" %}}
Make sure you went through all mesh-related parts of the previous 3 assignments
(at least up to the intermediate-level) before attempting to continue!

Typically, you should have attempted a number of meshing processes with
`blockMesh` at this point.
{{% /alert %}}

Welcome back to the third module in Part 1 of the "Reservoir engineering with
OpenFOAM" course.

Enough with the introductions and let's get to the real deal!!

The first step in OpenFOAM's simulation workflow which we are eager to learn is
the __meshing process__ (Let's forget about "Mathematical modeling of the
problem at hand" for now because it's really beyond the scope of this course).

In this module, we'll

- Discuss meshing for Reservoir Engineering tasks and meshing for FVM simulations.
- Assess mesh quality according to OpenFOAM metrics.
- Use `cfMesh` meshing workflows to mesh a pore-scale representation of a sample porous medium
- Learn about different meshing utilities (generators and manipulators) that are available in 
  Foam-Extend.

Let's get started.
