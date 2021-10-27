---
title: "02 - Manual Solution for the transport problem"
date: 2020-10-29
weight: 6
description: >
  Get a grasp of FVM simulations without using dedicated software. No heavy math, I promise.
math: true
---

Let's first start by solving the example transport problem using  amanual
approach.

## Mesh generation

The flow domain is meshed into 9 cells (labeled from 0 to 8) and we are
interested in $\phi$ values at each cell center.

![1D Mesh cells](/course/part-1/img/02.OpenFOAM-Simulation-in-a-nutshell/02-mesh-cells.png)

Notice that each cell has many faces and each face has a corresponing surface
normal vector.

![1D Mesh surface normals](/course/part-1/img/02.OpenFOAM-Simulation-in-a-nutshell/02-mesh-surface-normals.png)

> By a "surface normal vector", we mean a vector which is normal to the face
> and has the face's area as its module

To apply the finite volume method, it's important to have the boundary faces
point outwards. Note also that all faces that are "shared" between cells are
called __internal faces__.

OpenFOAM then labels these "cells" in a special way:

- A cell is said to be an __owner__ of a face if the face's normal is going
  __out of the cell__.
- A cell is said to be a __neighbor__ of a face if the face's normal is going
  __into the cell__.


{{% alert title="Example" color="success" %}}

![OpenFOAM owner and neighbor](/course/part-1/img/02.OpenFOAM-Simulation-in-a-nutshell/02-mesh-surface-normals.png)

- The owner of the face $f_{34}$ is cell `4`
- Its neighbor is cell `3`

> Note that Boundary faces have only owning cells

{{% /alert %}}

## Governing equation discretization

Now that the mesh is discretized properly, we need to discertize the flow
equation; mainly, applying it on each cell to derive a relationship of $\phi$
values at the cell center with its adjacent cells:

$$\nabla \cdot (\rho \mathbf{U} \phi) = \nabla \cdot (K \nabla \phi)
\qquad \longrightarrow \qquad \phi_c = f({\phi_{nearby\ cells}})$$

Throughout the development of this equation, we do make some assumptions:

- $\phi$ is considered to vary linearly between a cell center and a face
  center.
- The mesh must be constructed so that boundary faces are pointing __out__ of
  the domain.
- $\phi$ (value at cell center) is considered to be the __mean value__ of
  $\phi$ in that cell (with second order accuracy).
- `cellSize` has to be small enough for approximations to be accurate (to some
  extent).

- Write the governing equation in integral form for a single cell volume 
  ($V_c$):

$$\int_{V_c} \nabla \cdot (\rho \mathbf{U} \phi)\ \mathrm{d}V_c - 
\int_{V_c} \nabla \cdot (K \nabla \phi)\ \mathrm{d}V_c $$

- Use the divergence theorem: 
  $\int_{V_c} \nabla \phi\ \mathrm{d}V_c =
  \int{\partial V_c} \phi\ \mathrm{d}(\partial V_c)$ to convert volume integrals
  into surface ones. The transport equation then becomes:
  
$$\int_{\partial V_c} [\rho \mathbf{U} \phi - K \nabla \phi]\ 
\mathrm{d}(\partial V_c) = 0$$

> $\partial V_c$ denotes the closed, normal-outwarding, surface of the volume
> $V_c$

- If the closed surface $\partial V_c$ is split into a set of faces we can
  approximate the integral over the whole surface as a sum of face-based
  integrals:

$$\sum_{cell\ faces} [\int_{f} (\rho \mathbf{U} \phi)_f - 
\int_{f} (K \nabla \phi)_f ]\ \mathrm{d}\mathbf{S}_f= 0$$

> $\bar{\phi_f}$ is the mean value of $\phi$ over the face $f$

- We can then introduce the relationship of the mean value over the face:
  $|\mathbf{S}_f|\bar{\phi_f} = \int_f \phi\ \mathrm{d}\mathbf{S}_f$
  to produce the following discrete equation:

$$\sum_{cell\ faces} [\overline{(\rho \mathbf{U} \phi)}_f -
\overline{(K \nabla \phi)}_f]|\mathbf{S}_f| = 0$$

## Obtaining the system of algrebraic equations

- Apply the previous equation to all 9 cells.
- Considering $K = 0.01$ and $\rho \mathbf{U} = \begin{bmatrix} 0.03 \\ 0 \\ 0 \end{bmatrix}$

But the previous equation relates __face values__ to each other, and we are
interested in __cell values__ instead. That's why there is a need for defining
a scheme to convert face values to cell-centered ones (an interpolation scheme).

Assuming a linear evolution, a second-order accurate Central Difference Scheme
can do the job (Example for cell 4):

$$\phi_{f_{45}} = \phi_4 + \frac{\phi_5 - \phi_4}{c_{4}-c_{5}}
(x_{f_{45}} - c_{4})$$

> `c_i` is the position of cell $i$

Which can be further simplified to an average because of the uniform grid
spacing:

$$\rho_{f_{45}} = \frac{\phi_5 + \phi_4}{2}$$

- It remains to process the gradients in the face-normal direction for us to
  get to the final discrete equation

$$[(\rho u \phi)_{f45}  -(K \frac{d\ \phi}{dx})_{f45}] -
[(\rho u \phi)_{f34}  -(K \frac{d\ \phi}{dx})_{f34}] = 0$$

- Which can be simplified to (Remember, this equation is for cell 4):
$$-0.115\ \phi_3 + 0.2\ \phi_4 - 0.085\ \phi_5 = 0$$

- For the boundary faces, at least the __value__ or the __gradient__ of $\phi$
  has to be supplied.
  - For our purpose, set $\phi_0 = 1$ and $\phi_9 = 0$

The system for all 9 cells should then look like this:

$$
\begin{bmatrix}
    0.315\phi_0 & - 0.085\phi_1 & & & & & & & & = 0.23\\
    -0.115\phi_0 & + 0.2\phi_1 & - 0.085\phi_2          & & & & & & & = 0\\
    & -0.115\phi_1 & + 0.2\phi_2 & - 0.085\phi_3        & & & & & & = 0\\
    & & -0.115\phi_2 & + 0.2\phi_3 & - 0.085\phi_4      & & & & & = 0\\
    & & & -0.115\phi_3 & + 0.2\phi_4 & - 0.085\phi_5    & & & & = 0\\
    & & & & -0.115\phi_4 & + 0.2\phi_5 & - 0.085\phi_6  & & & = 0\\
    & & & & & -0.115\phi_5 & + 0.2\phi_6 &- 0.085\phi_7 & & =0\\
    & & & & & & -0.115\phi_6 & + 0.2\phi_7 & - 0.085\phi_8 &=0\\
    & & & & & & & -0.115\phi_7 & + 0.285\phi_8 & = 0  
\end{bmatrix}
$$

### Solving the system of equations

For simplicity, I have opted for the __Gauss-Seidel__ iterative method for
solving the resulting system of equation (although the direct approach is
sufficient in this case):

- Start with an initial solution vector for $\phi$ values
- Update the solution of $A \phi = B$ for the new iteration ($k+1$) with:
  $$\phi_i^{(k+1)} = \frac{1}{a_{ii}} (b_i -\sum_{j=1}^{i-1}a_{ij}\phi_j^{(k+1)}
  -\sum_{j=i+1}^{n}a_{ij}\phi_j^{(k)} )$$
  Where $A = [a_{ij}],\ B = [b_i]$

We can then obtain the final steady-state solution ($\phi$ values at cell
centers):

| Iteration | Cell 0 | Cell 1 | Cell 2 | Cell 3 | Cell 4 | Cell 5 | Cell 6 | Cell 7 | Cell 8 |
|-----------|--------|--------|--------|--------|--------|--------|--------|--------|--------|
| 00 |        0 |       0 |       0 |     0   |    0    |       0 |      0 |       0 |     0 |
| 01 |  0.7301  | 0.4198  | 0.2414  | 0.1388  | 0.0798  | 0.0458  |0.0263  | 0.0151  | 0.0061|
| 02 |  0.8434  | 0.5875  | 0.3968  | 0.2621  |  0.1702 | 0.1090  |0.0691  | 0.0423  | 0.0171|
| 03 |  0.8887  | 0.6796  | 0.5022  | 0.3611  | 0.2540  | 0.1754  |0.1188  | 0.0756  | 0.0305|
| 04 |  0.9135  | 0.7387  | 0.5782  | 0.4404  | 0.3278  | 0.2390  |0.1695  | 0.1104  | 0.0445|
| 05 |  0.9295  |  0.7802 | 0.6358  | 0.5049  | 0.3919  | 0.2974  |0.2179  | 0.1442  | 0.0582|
| ...|  ...     |    ...  | ...     | ...     | ...     | ...     |...     | ...     | ...   |
| 35 |  0.9888  | 0.9588  | 0.9188  | 0.8653  | 0.7938  | 0.6978  |0.5687  | 0.3946  | 0.1592|
| 36 |  0.9888  | 0.9591  | 0.9192  | 0.8659  | 0.7945  | 0.6985  |0.5694  | 0.3950  | 0.1594|
| 37 |  0.9889  | 0.9593  | 0.9196  | 0.8664  | 0.7951  | 0.6991  |0.5699  | 0.3954  | 0.1595|

For a tolerance of `1e-3`, we can say that the method converges in around 37
iterations.

> Notice that the initial $\phi$ values were all zeros

## Learn more

- Face gradients are evaluated in the direction of the face normal.
- The flow is unidirectional, so technically, the governing equation is a 2nd order ODE. 
- See **Related Files** section for a PDF file explaining the steps to acquire the theoretical
  and manual solutions.
- From here, you can head strait to the next unit.

{{% attachments title="Related files" pattern=".*pdf" %}}
