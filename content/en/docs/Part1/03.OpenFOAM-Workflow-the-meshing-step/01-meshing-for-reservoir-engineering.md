---
title: "01 - Meshing for Reservoir Engineering tasks"
date: 2020-11-15
weight: 5
description: >
  Common meshing practices in Reservoir Engineering and their relation to OpenFOAM-based meshes.
mermaid: true
math: true
---

## An introduction to OpenFOAM meshing

### A quick note about OpenFOAM Mesh Regions

You should already know that, by default, OpenFOAM mesh data is stored under
`constant/polyMesh` directory in each case. In fact, __polyMesh__ here refers to
the mesh "region name". So, if you have multiple mesh regions (region0, region1)
, their data will be stored in `constant/region0`, `constant/region1`
respectively.

These mesh regions can be totally isolated, or coupled through some coupled
interfces, depending on the solver to be used. Of course, each region will
require initial and boundary conditions for its fields.

The rest of the unit assumes we are working on the default single-__polyMesh__
region, but everything applies also to multi-region meshes.

### OpenFOAM Mesh files

The following diagram briefly presents OpenFOAM's basic mesh files with the most
important notes about the contents of each file:

{{<mermaid align="center">}}
graph LR;
    A["constant/polyMesh"];
    style A fill:orange,stroke:red

    A --> B(fa:fa-grip-vertical points);
    A --> F(fa:fa-square-full faces);
    A --> C(fa:fa-vector-square boundary);
    A --> D(fa:fa-level-down-alt owner);
    A --> E(fa:fa-level-up-alt neighbor);


    subgraph "Faces Information"
    F --- Q("Connections between vertices as faces<br>Each face is a list of
    connected vertex indices<br>The order of vertices provides a manifold
    representation of the face")

    C --- K("Boundary patches<br>Boundary faces are grouped into <b>patches</b>
    of different types")

    D --- M("Owner cells for faces<br> Downstream cell index according
    to face normals is associated<br> with each internal and boundary face")

    E --- O("Neighbor cells for internal faces<br> Upstream cell index
    according to face normals is associated<br> with each internal face<br>
    Boundary faces have only owner cells!")
    end

    subgraph "Points Information"
    B --- I("Mesh Vertex 3D-Coordinates<br>Each vertex gains an <b>index</b> in
    the order it appears in<br>No vertex duplication allowed")
    end

    style I fill:lightgreen,stroke:lightgreen
    style Q fill:lightgreen,stroke:lightgreen
    style K fill:lightgreen,stroke:lightgreen
    style M fill:lightgreen,stroke:lightgreen
    style O fill:lightgreen,stroke:lightgreen

{{< /mermaid >}}

{{% alert title="Example" color="success" %}}
Assume `constant/polyMesh/points` starts with:
```cpp
2142        // Number of mesh points
(
(0 0 0)     // Vertex 0
(0.5 0 0)   // Vertex 1
(1 0 0)     // Vertex 2
.....
(25 10 0.1) // Vertex 2141
);
```

A face list can then specify any type of faces:
```cpp
4071              // Number of mesh faces
(
4(1 52 1123 1072) // face 0 from vertices 1, 52, 1123 and 1072
.....
3(1018 1069 1070) // face 4069 is a triangle
3(1018 1070 1019) // face 4070
);
```

And each face will have at least an owner cell
(and potentially a neighbor cell):
```cpp
4071              // Number of mesh faces
(
0                 // face 0 has cell 0 as its owner
0                 // face 1 also has cell 0 as its owner
.....
);
```
{{% /alert %}}

## Meshing workflow with `blockMesh`

Up until now, we have interacted with only a single mesh generation tool:
`blockMesh`, let's review some concepts to keep in mind while composing the
`blockMeshDict`:

### Fundamentals of meshing with `blockMesh` utility

1. The mesh is built out of "hex blocks", which allows for each block to have 
   cell size and grading settings.
   - Naturally, because these blocks will be next to each other, shared faces
   must be consistent (Watch out for grading problems!)
   - If two blocks are to share a face, __they at least share one full edge__.
   This is especially important because in the hex block specification, you're
   actually allowed to duplicate point indices to produce a wedge shape!

2. Always run `checkMesh` right after mesh generation.

3. It's hard to blindly choose vertex indices to use in `blockMeshDict.blocks`,
   but, fortunately, there are some tools to help us with this issue:
   - `paraFoam -block` will show a representation of the contents of
     `blockMeshDict` and not the generated mesh (if this utility is available).
   - `pyFoamDisplayBlockMesh.py` (from PyFoam) will do the same job.

For Reservoir Enginnering tasks:

- Complex block-structured meshes are possible (and relatively easy actually)
  to achieve by using a __Macro Language__ (Eg. M4) to automate the generation
  of complex `blockMeshDict` files.

- For complex geometries, it's recommended to prefer (in this order):
  - Mesh conversion tools (from VTK, Salome, GMSH ... etc).
  - `cfMesh` for easy-n-fast mesh genration from geometry files (Eg. STL).
  - `snappyHexMesh` as a last resort for greater control over the mesh.

- Again, always check your mesh's quality right after each mesh generation
  process!

### How to choose the cell size

> Now, let's get away from all the utilities and discuss the actual meshing
> concept.

The question is: __what cell size should we choose for our domain?__

It's going to depend on the specific simulated problem, but the rule is:

- __Finer mesh__ in flow-critical regions of the domain
- Coarser mesh can be __tolerated__ in less important regions

but __How fine the flow-critical regions should be?__

There is no general rule but:

- Too-small cell size introduces numerical instability
- The more you increase it, the __even more__ accuracy will be lost.
- In transient simulations, mesh size is closly related to time step length.
  - It's common practice to fix the mesh size, then decide on a good 
    $\Delta t$

To illustrate, let's go through a simplified problem for seismic waves.

These waves can propagate at a speed of 3 km/s, with a maximum frequency on 1Hz:

![Visualization of Global Seismic Wave Propagation
Simulation](/course/part-1/img/03.OpenFOAM-Workflow-the-meshing-step/01-seismic-waves.gif)

From $c = \lambda f$, we deduce that the wave's length
$\lambda = 3 \mathrm{m}$. Now, let's assume we have a PDE modeling how such
waves propagate; and the domain on which the study is performed is the 
__"whole planet Earth"__.

In order to get (somewhat) accurate results, say:

- We consider 10 grid cells __per wave length__
- This leads to a cell size of 300 m (also assume Hexagonal cells).

Now we face a number of problems:


{{% alert title="Problem 1" color="danger" %}}

Recall that the wave's propagation speed is $c = 3 \mathrm{km.s^{-1}}$

- After __1 second__, the wave would have passed by __10 grid cells__
  - This will surely generate continuity errors and all sorts of numerical
  instabilities.

{{% /alert %}}


{{% alert title="Solution to Problem 1" color="primary" %}}

Simply put, choose a smaller time step length

- $\Delta t = 0.1 \mathrm{s}$ seems a reasonable maximum value

{{% /alert %}}

{{% alert title="Problem 2" color="danger" %}}

Cell volumes would have the value of $V_c = 0.3^3 \mathrm{km^3}$

- Earch Volume: $V_E = \frac{4}{3} \pi (6371)^3 \mathrm{km^3}$
- Number of cells needed to mesh the whole earth:
  $\frac{V_E}{V_c} \approx 4 \times 10^{13}$
  - So, for __each__ scalar field (assuming C's double), we would need around
  __320 TeraBytes__ as storage space (RAM & Hard Disk)!

{{% /alert %}}

{{% alert title="Solution to Problem 2" color="primary" %}}

Even __10 grid cells per wave length__ doesn't seem achieveable now, huh?

- Coarsen the mesh and go back to Problem 1

{{% /alert %}}

### Meshing complex geometries for Reservoir Engineering

- Tools to convert mesh from industry-standard formats (Eg. ECLIPSE's Corner
grids) are currently under development!

Thus, we have two options:

1. Either convert to-and-from VTK as intermediate step
    - Works well if the original grid is mostly compatible with OpenFOAM mesh
      standards
    - Need to re-select faults faces into `faceSets` after conversion

2. Use `snappyHexMesh` or `cfMesh` on the original `geometry`
    - Produces very similar, but not identical, meshes (to the original grid)
    - Permeability Upscaling may pose a problem because cells do not have the
      same topology as the original grid!

## Learn more

- The 
  [Visualization of Global Seismic Wave Propagation Simulation](https://commons.wikimedia.org/wiki/File:Global_Seismic_Wave_Propagation_Simulation.gif)
  is created by Greg Abram and released under the Creative Commons Attribution
  4.0 International license.
- Storage calculations are based on C's `double` type on a 64-bits machines,
  which is a typical setup in today's simulations.
- Head to this 
  [quick assignment to practice near-well meshing with `blockMesh`](https://classroom.github.com/a/rWU_VhMC)
  before continuing to the next unit.
