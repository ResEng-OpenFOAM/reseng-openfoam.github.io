---
title: "03 - Meshing Utilities"
date: 2020-11-19
weight: 7
description: >
  Generating and manupulating quality meshes with the least effort possible using different meshing tools!
mermaid: true
math: true
---

In this unit, we'll take a quick look on some of the most important OpenFOAM
meshing utilities besides `blockMesh`

## cfMesh

`cfMesh` is not actually an official OpenFOAM utility, it's a set of utilties
and libraries built on top of OpenFOAM.

- It is developed by Creative Fields Ltd.
  - Hosted at their [cfmesh.com/cfmesh](https://cfmesh.com/cfmesh)

- It requires only
  - A __geometry file__ (in .STL, .FTR, or .FMS format).
  - and a (short) dictionary file (`system/meshDict`) specifying meshing settings.


The typical workflow a chMesh utility involves:

1. Start from a basic auto-generated hex-based mesh
2. Incrementally improve it until it meats user requirementsusing specific
   meshing workflows:
   - __cartesianMesh:__ for 2D hex meshes
   - __cartesianMesh, tetMesh, pMesh:__ for 3D hex-based, tetrahedral and
     polyhedral meshes respectively

This process is controled by some settings mentioned in `system/meshDict`, which
must contain at least:
   - A `surfaceFile` entry, pointing to the geometry to be meshed
   - `maxCellSize`

### Meshing example

{{% alert title="Example" color="success" %}}

Let's illustrate by going through a meshing process of a pore-scale
representation of a 2D porous medium:

- The first step is to grab 2D images for the domain (which can also be
  artificially generated using the __PoresPy__ library - More on this in the
  assignment)

  ![](/course/part-1/img/03.OpenFOAM-Workflow-the-meshing-step/03-porous-medium.png)

  The yellow region represents the grains and the medium's porosity is roughly
  0.6

- Then, we can use somem image processing software to extract SVG paths defining
  grains boundaries, which can be later extruded using a CAD software to produce
  an STL surface as shown is the following figure:

  ![](/course/part-1/img/03.OpenFOAM-Workflow-the-meshing-step/03-porous-medium-stl.png)

- `cartesian2DMesh` requiresthat no faces with normals in the z-direction are
  present in the surface file, so, we remove those faces from our surface:

  ![](/course/part-1/img/03.OpenFOAM-Workflow-the-meshing-step/03-porous-medium-2d.png)

- The `system/meshDict` should then look like this:
  ```cpp
  surfaceFile "porousMedia2D.stl";
  minCellSize 0.5;
  maxCellSize 3.5;
  ```

- All that remains is to actually running the meshing command:
  ```bash
  (of@case) cartesian2DMesh | tee log.mesh
  ```

- To finally produce something like:

  ![](/course/part-1/img/03.OpenFOAM-Workflow-the-meshing-step/03-porous-medium-mesh.png)

{{% /alert %}}


### cfMesh workflow steps

- If the surface has some patches defined, they are automatically picked up 
  - If not, you have to define them as OpenFOAM boundary patches

- Also, the same dictionary file can be used for different meshing workflows
  - Possibly with some cell size adjustments

You can control at which workflow step the meshing utility stops using an
entry in `system/meshDict` (You can also resume from previous meshing
operations):

```cpp
workflowControls
{
    stopAfter templateGeneration;
    restartFromLatestStep 1;
}
```

The previous settings will cause the meshing command to stop right after
generating the base mesh; which is a hex-based mesh surrounding the input
geometry. The `minCellSize` setting greatly affects cell size at this stage.


Another useful step is __surfaceProjection__, where mesh points are projected on
the provided surface. I usually stop here to check how well the mesh is
projected on a low-quality surface.

> Check cfMesh's User guide for a list of available workflow controls

## snappyHexMesh

`snappyHexMesh` is the official utility for generating complex Hex-based meshes

- Works on an __existing (user-supplied)__ mesh, so it’s perfect for modifying
  meshes.
- The utility then snaps mesh vertices to a surface (STL or OBJ file)
- If the user requires it, boundary layers are generated as the last step

Some common features with `cfMesh`:

- Parallel meshing
- Local mesh refinement
- Similar workflow controls
- But it is driven by a lengthy dictionary file `system/snappyHexMeshDict`

The utility performs three actions in order:


{{<mermaid align="center">}}
graph LR;

    subgraph "boundaryLayer [optional]"

    F("Additional number of cells <br>added in layers near wall patches") 

    end

    subgraph "snapMesh [optional]"

    D("Align mesh vertices<br>with the geometry") --- E("Hex-cells get transformed to polyhedrons,<br>merged and/or deleted")

    end


    subgraph "castellatedMesh"

    A("Delete everything outside<br>of the flow domain") --- B("Local mesh refinements")

    B --- C("Produces a fully hex-based mesh,<br>similar to ECLIPSE-Like Grids")

    end

{{< /mermaid >}}

-----

Surface (generally Stereo-lithography .STL, Nastran .NAS or .OBJ) files are 
looked up in `constant/triSurface` directory of the case.

- __ALWAYS check the geometry file__ before doing anything else. You can simply 
  use `surfaceCheck` for this (an OpenFOAM utility).

- Common CAD geometry problems you should be concerned with include:
    - Overlapping triangles (A serious one, absolutely fix these before meshing)
    - Open geometries (where there is no "inside" and "outside"; tools will get
      confused about which regions to mesh).

With `snappyHexMesh`, you get one huge advantage:

- __Custom mesh quality metrics__ can be provided in a separate dictionary file
  to use during mesh generation (The resulting mesh will meet those
  requirements)!


### Mesh refinement in `snappyHexMesh`

Mesh refinement is measured in "Levels":

- Level 0: the original mesh refinement
- Level 1: splits each cell to four (`$2^2$`) cells
- Level 2: splits each cell to eight (`$2^3$`) cells and so on

Each time we increase the refinement level, the edges are __halved__! This is,
of course, valid only for hexahedrons (That's why `snappyHexMesh` needs a fully
hex-based background mesh).

## Miscellaneous mesh tools

OpenFOAM also provides some mesh manipulation tools:

- __autoPatch__: which automatically extracts boundary patches based on a feature
  angle.
- __cellSet, faceSet__: which create and manipulate cell and face "groups"
  respectively.
- __transformPoints__: which moves, rotates and/or scales the mesh in any
  direction

The next set of tools OpenFOAM provides is the conversion tools; `gmshToFoam`
and `ansysToFoam` being the most popular ones.

And lastly, there are some advanced tools for even finer control over the mesh.
Examples include __refineHexMesh__, __splitMesh__ and __collapseMesh__.



## Learn more

- There is a 
  [Comprehensive Tour of `snappyHexMesh`](https://openfoamwiki.net/images/f/f0/Final-AndrewJacksonSlidesOFW7.pdf)
  , created by Engys Ltd. (Specifically: Andrew Jackson), so I see no point in
  replicating what they did there! **When** you absolutely need to learn
  how to work with `snappyHexMesh`, use their slides!!
- You can skim through this 
  [`cfMesh` User Guide](http://cfmesh.com/wp-content/uploads/2015/09/User_Guide-cfMesh_v1.1.pdf)
  to get a grasp of what's available.
- At this stage, It's recommended that you go through the detailed 
  [Meshing Tools Individual Assignment](https://classroom.github.com/a/2fPp0s4c) 
  to gain some hands-on experience.

## Oh! There is a group assignment

To conclude this module, you are asked to participate in the **hunt**
for the best mesh, which must be __produced__ using OpenFOAM utilities
(`blockMesh`, `cfMesh`, `snappyHexMesh`, NO conversions), for a well-known
domain in your field of study.

### For Reservoir Engineering practitioners

Each group should pick a domain to mesh (we are **only** interested in the mesh
here) from the following list of cases:

- [The Egg Model](http://www.mufits.imec.msu.ru/example-egg-model.html) 
  from MUFITS examples page.
- [The Optimal well placement case](http://www.mufits.imec.msu.ru/example-well-placement.html) 
  from MUFITS examples page.
- [The Benchmark study for CO2 storage](http://www.mufits.imec.msu.ru/example-example-h3.html)
  from MUFITS examples page.
- [The Norne Field case](https://opm-project.org/?page_id=559)
  from OPM Open Datasets.

### For Other fields

- You are free to choose any geometry for any well-known problem in your field
  and mesh it using OpenFOAM utilities. The only requirement is that your
  meshing shouldn't be easy enough for a **one newcomer** to OpenFOAM to build
  quickly. In short, look for challenging mesh requirements.

Follow this [Github Classroom](https://classroom.github.com/g/s1bNX2JV)
link to participate, get your meshing files and mesh stats into your repo
and wait for feedback and improvements from your team :)
