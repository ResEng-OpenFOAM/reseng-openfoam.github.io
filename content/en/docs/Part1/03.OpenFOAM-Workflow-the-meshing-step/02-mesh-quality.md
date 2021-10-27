---
title: "02 - OpenFOAM Mesh Quality"
date: 2020-11-17
weight: 6
description: >
  Discussing what makes a mesh a "good mesh" from OpenFOAM's perspective
math: true
mermaid: true
---

We have seen that, more often than not, it's not feasible to use perfectly
dense mesh. So, we are __forced__ fall back to a coarser one; where __its
quality__ becomes an important factor:

- Our approximations can easily go wrong on degenerate meshes
- You lose some "solution accuracy" while coarsening the mesh
- And you may even lose some efficiency because of the attempts to compensate
  for lost accuracy by doing some corrections!

That's why (almost) all CFD packages have some __mesh quality standards__.

In OpenFOAM context, for our simulations to converge to correct solution, we
at least need a "good" mesh:

    - Which is associated with accpetable errors
    - Passes the checks made by `checkMesh` utility which evaluates the mesh's
      quality using a set of metrics.

## Quality metrics for `checkMesh`

### Non-Orthogonality

Non-Orthogonality is an angle measuring how slated the face normal is relative
to the vector between cell centers. This angle is especially important when we
approximate gradients in the direction of cell centers by the gradient
calculated in the __face normal__ (most commonly in the diffusion operator).

These approximations can seriously go wrong if the maximal non-orthogonality
reaches 70 degrees for hexagonal meshes (and around 80 in tetrahedral ones).

> In the case of boundary faces, we just use the face center.

![](/course/part-1/img/03.OpenFOAM-Workflow-the-meshing-step/02-nonorthogonality.png)

### Skewness

The skewness is a measure of how far the face center is placed in respect to
the intersection of the vector connecting cell centers with the face, normalized
against the distance between cell centers.

The rule of thumb is to have a max skewness of (at most) 4. If skewed faces
present a very small percentage of total face numbers (say < 0.01%) and are
randomly distributed, it's probably OK even if the skewness reaches 4 and
beyond. But, if they are clustered around some mesh feature (eg. a locally
refined region which is important to the flow), it's recommended to fix them!
Especially if some divergence schemes are involved.

> In the case of boundary faces, we use the orthogonal projection of the cell
> center on the face instead of the intersection point.

![](/course/part-1/img/03.OpenFOAM-Workflow-the-meshing-step/02-skewness.png)

### Aspect Ratio

This metric expresses the ratio of the largest __cell edge__ to the smallest
one, thus, it might be useful to consider the reciprocal of this number as it
can become very big.

Assume a triangle ABC, where C can move freely on a half-plane:

![](/course/part-1/img/03.OpenFOAM-Workflow-the-meshing-step/02-aspectratio.png)

You can see that the optimum position of C is where the triangle becomes an
equilateral one (aspect ratio of 1).

This metric is particularly important if the aspect ratio is increasing rapidly
in a single mesh direction (which will cause interpolation schemes to
artificially favorite cells with larger volumes and lead to interpolation
errors)

### Face convexity

To understand the importance of this metric, we first need to review how faces
are treated in OpenFOAM:

#### Triangle faces in OpenFOAM

Computationally, __triangles__ are the only 2D geometries with
directly-determined surface area (because they are the simplest).

- If the surface of any triangle $\mathrm{A_0A_1A_2}$ where
  $\mathrm{A_m}\ (\mathrm{x_m, y_m, z_m}),\ \mathrm{m} = 0,1,2$ is needed:
  $S=\frac{1}{2}|\mathbf{A_0A_1} \times \mathbf{A_0A_2}|=\frac{1}{2} 
            \begin{vmatrix}
                \mathbf{i} & \mathbf{j} & \mathbf{k} \\
                x_1-x_0 & y_1-y_0 & z_1-z_2 \\
                x_2-x_0 & y_2-y_0 & z_2-z_2 \\
            \end{vmatrix}$
- Then, the area of a quad can be calculated using two triangles
- Similarly,  a __tetrahedron__ volume is obtained directly and it's used to 
  determine the volume of pentahedrons (three tets) and hexahedrons 
  (five or six tets)!

This makes tetrahedral meshes the most efficient ones when it comes to
mesh-related calculations.

#### Quad faces in OpenFOAM

Another popular mesh type is the hexahedral one. OpenFOAM treats the quad faces
of such cells in (roughly) the following way:

- Figure out the __4__ triangles formed by the quad's diagonal and two the
   quads edges and then compute their areas in a specified direction (Let's
   assume the anti-clockwise way):

    ![](/course/part-1/img/03.OpenFOAM-Workflow-the-meshing-step/02-quad01.png)
    ![](/course/part-1/img/03.OpenFOAM-Workflow-the-meshing-step/02-quad02.png)

- If all 4 areas are positive, the quad is convex.
- If one of these areas is null, the quad is degenerate (triangle 123):

    ![](/course/part-1/img/03.OpenFOAM-Workflow-the-meshing-step/02-quad03.png)

- If exactly one of these areas is negative (again, the triangle 123):

    ![](/course/part-1/img/03.OpenFOAM-Workflow-the-meshing-step/02-quad04.png)

- If two of these areas the quad is either a self-intersection, or a negative
  one:

    ![](/course/part-1/img/03.OpenFOAM-Workflow-the-meshing-step/02-quad05.png)
    ![](/course/part-1/img/03.OpenFOAM-Workflow-the-meshing-step/02-quad06.png)

In short, OpenFOAM does not allow __self-intersecting__ quads and may fix the
rest to obtain only convex faces.

## `checkMesh` check types

Let's now move on to the checks performed by `checkMesh` utility.

### Topology checks

The best way to present these checks is by using a diagram:

{{<mermaid align="center">}}
graph LR;

    subgraph "if -allTopology option is supplied"

    E("Over-Used edges") --- o("Presence of edges used by too-many cells")
    F("Boundary cells") --- n("Cells with 0 or 1 'internal' faces<br>
    (Isolated cells, cells with too many boundary faces)")
    G("Non-Manifold points") --- p("Vertices which participate in cell
    definition<br>in a non-manifold way")
    end

    A("Boundary Definition") --- q("Things like:<br>
    Whether boundary faces belong exclusively to patches<br>
    Wether boundary patches are correctly defined")

    B("Illegal Cells") --- k("Cells with less than 4 faces<br>
    Cells with out-of-range faces!")

    C("Unused points<br>Unordered faces") --- m("Vertices defined but not used
    in faces definition<br>and incorrectly-ordered faces")

    style q fill:lightgreen,stroke:lightgreen
    style k fill:lightgreen,stroke:lightgreen
    style m fill:lightgreen,stroke:lightgreen
    style o fill:lightgreen,stroke:lightgreen
    style n fill:lightgreen,stroke:lightgreen
    style p fill:lightgreen,stroke:lightgreen

{{< /mermaid >}}

### Geometry checks

{{<mermaid align="center">}}
graph LR;

    subgraph "if -allGeometry option is supplied"
    H("Points on short edges") --- K("Warped faces") 
    K --- L("Under-determined cells")
    R("Faces and cells with<br>low interpolation weights")
    end

    A("Bounding box") --- B("Smallest Cell Size")
    B --- C("Geometrical &<br>Solution directions")

    D("Non-Closed Cless &<br>Aspect Ratio") --- E("Zero-Area Faces<br>Zero-Volume Cells")
    F("NonOrthogonality &<br>Skewness") --- G("Face Orientation")

{{< /mermaid >}}

## Learn more

- Each software package has (should have) its own "set of mesh quality
  standards", based on the numerical methods used to solve the equations.
  The metrics explained here along with their "calculation methods" are
  **OpenFOAM-specific**.
- If you want a **second-order**-accurate FVM simulations (which are 
  usually hard to achieve when it comes to reservoir simulations), stick
  with the following metrics:
  - For hex-based meshes:
      * Maximal Non-Orthogonality < 70 degrees
	  * Maximal skewness < 4  for internal faces
	  * Maximal skewness < 20 for boundary faces
  - For tetrahedral meshes, you can allow up to 80 degrees as the maximal
  Non-Orthogonality.
