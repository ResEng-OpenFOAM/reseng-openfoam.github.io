---
title: "04 - Result visualization with ParaView"
date: 2020-11-11
weight: 8
description: >
  Basic data visualization and manipulation with the ParaView package
math: true
---

In the previous two units, we have solved our sample transport problem both
manually and with the help of OpenFOAM; It's time to learn how to properly
visualize the results of such simulations.

> For this particular case (1D mesh), a line plot of the main variable would
> suffice; but we will dig a little deeper into ParaView's workflow to prepare
> for more complex mesh-based visualizations.

## ParaView UI Basics

This not a comprehensive ParaView tutorial, but you need to know ParaView's UI
parts so you can use them efficiently from the get-go:

![ParaView UI elements](/course/part-1/img/02.OpenFOAM-Simulation-in-a-nutshell/04-paraview-ui.png)

- The rendering window (Which points to a __3D View__ of the mesh in the
  picture) is where you "see" stuff. It can be split into multiple views of a
  lot of things (Most useful views are: Rendering view, spreadcheet view, and
  plot views).

- The __menu bar__ is where all the tools, filters and sources are located.
- The __tool bar__ handles variable coloring, mesh display mode and some camera
  settings.
- The __pipeline browser__ shows and activates visualization objects.
- And each object's properties can be inspected and modified in the 
  __properties panel__


The standard workflow for visualizing data can be summerized as:

1. __Reading data into the software:__ which is usually done through a "Reader
   module". For example, there is a default OpenFOAM Reader in ParaView which
   recognizes OpenFOAM case directories if an empty `*.foam` file is loaded.
   - You can also export OpenFOAM data as VTK files (ParaView's native format)
     and then use the VTK reader to load it.
   - OpenFOAM has its own ParaView Reader! If you're using your local machine
     to follow along, try running `paraFoam` inside a case.
2. __Filtering:__ which typically involves data manipulation and modification
   using a series of "filters".
3. __Rendering:__ which results in presentation-ready images or video-scenes of
   data features.

### Visualization with VTK (for the manual approach)

Let's start by visualizing the data we got from the manual solution, as if we
never simulated the problem with OpenFOAM.

As mentioned previously, we know cell center positions, and we know `T` values
at these positions, so, a line plot is the best visualization option; but let's
not do that. Instead, let's take advantage of ParaView's native data format
(VTK) to visualize the data on a __mesh__. 

The (legacy-) VTK file has a simple format:

```cpp
# vtk DataFile Version 3.0
Manual visualization
ASCII
DATASET RECTILINEAR_GRID
DIMENSIONS 10 2 2 
X_COORDINATES 10 float
0.0 0.1 0.2 0.3 0.4 0.5 0.6 0.7 0.8 0.9 
Y_COORDINATES 2 float
0 0.1
Z_COORDINATES 2 float
0 0.1
CELL_DATA 9
SCALARS T float 1
LOOKUP_TABLE default
0.98896731 0.95934624 0.91965778
0.86647731 0.79512318 0.69919388
0.56994799 0.39547386 0.15957717
```

- The first line is a comment, explaining what the file is
- The second line holds a title for the visualization
- The third line specifies whether the data is in binary of ascii format
- The forth line selects the grid type. In this case, we're visualizing data on
  a rectilinear grid (which is a structured type, but doesn't imply uniform
  cell size; more in the [User Guide](https://docs.paraview.org/en/latest/UsersGuide/understandingData.html)).
- Next, the number of __mesh points__ in each of the three directions is
  specified with the "DIMENSIONS" keyword.
- The next 6 lines specify coordinates for these points per direction. Note that
  these coordinates are for __mesh vertices__, not for cell centers.
- The keyword `CELL_DATA` starts the specification of 9 cell-centered data.
- We have a single scalar field `T` (of "float" type)
- This `T` field is read from a lookup table (in the default format) right after
  the `SCALARS` line.

> This file is built so that we keep the previous numbering of mesh cells.

Now, let's go through the process of visualizing the data in this file one step
at a time:

- First, load the file into ParaView (You may need to click on some "Apply"
  buttons):
  ![ParaView UI elements](/course/part-1/img/02.OpenFOAM-Simulation-in-a-nutshell/05-paraview-read-data.png)
- By default, the loaded object is selected in the pipeline browser but has an
  "Outline" display mode:
  ![ParaView UI elements](/course/part-1/img/02.OpenFOAM-Simulation-in-a-nutshell/05-paraview-pipeline.png)
- Let's flip the display mode to say `Surface with edges` and color the surface
  with `T` values:
  ![ParaView UI elements](/course/part-1/img/02.OpenFOAM-Simulation-in-a-nutshell/05-paraview-coloring.png)
- The box in front of `T` in the following picture indicates that the field is
  __cell-based__, but for a smoother visualization, let's interpolate it to mesh
  points. To do that, pring up the filters search with `Ctrl+Space` and type
  `Cell data To Point Data`.
  ![ParaView UI elements](/course/part-1/img/02.OpenFOAM-Simulation-in-a-nutshell/05-paraview-filter1.png)

- Now (ater hitting the apply button), a new object is selected in the pipeline
  browser, and there is a "dot" in front of `T` (Indicating a points-based
  field).
  ![ParaView UI elements](/course/part-1/img/02.OpenFOAM-Simulation-in-a-nutshell/05-paraview-filter2.png)

- An important observation is that while `T` 's range is correct for the cell
  data, the point data should account for boundary values. In our case, we
  expect `T` to vary between 0 and 1. Fixing this issue is not hard:
  - Save the points field as a VTK file (in ascii format)
  - Change all occurances of `0.988967` to 1 and all occurances of `0.159577` to 0
  - Then re-read the modified file!

- Now, we're finally ready to produce that line plot you're eager to see. With
  the loaded file selected, apply the `Plot Over Line` filter:
  ![ParaView UI elements](/course/part-1/img/02.OpenFOAM-Simulation-in-a-nutshell/05-paraview-filter3.png)

- But the line we're plotting `T` over should be on the x-axis (Use the
  properties panel to modify this property of the line):
  ![ParaView UI elements](/course/part-1/img/02.OpenFOAM-Simulation-in-a-nutshell/05-paraview-filter4.png)

- This filter will open a new view, containing the plot; and you can continue
  modifying the filter's properties.
  ![ParaView UI elements](/course/part-1/img/02.OpenFOAM-Simulation-in-a-nutshell/05-paraview-filter5.png)

### Visualization with VTK (for OpenFOAM's case)

Visualizing OpenFOAM results is basically the same,
the only difference is in the way we load case data into ParaView:

1. If OpenFOAM is compiled with ParaView support, the best way is
   to run `paraFoam` inside the case's directory. Unfortunately, this is not the
   case if you're using the LinuxOne lab machine.
2. If ParaView is not installed on the same machine as OpenFOAM, but has access
   to the case directory, run `touch casename.foam && paraview casename.foam`
3. You can also convert OpenFOAM's data to VTK then read it into ParaView:
   `foamToVTK --ascii` (Then the files will be in the `case/VTK` directory).

## Final Discussion

What about comparing our results with theoretical solutions?

`$$T(x) = \frac{1}{\rho |\mathbf{U}|}(aK e^{\frac{\rho |\mathbf{U}|}{K} x}+b)$$`

Where

- `$K = 0.01,\quad \rho |\mathbf{U}| = 0.03$`
- `$a = -0.21614$` and `$b = 0.0321614$`, deduced from boundary conditions

OpenFOAM Solutions are __so accurate__ because we used the __Central Difference
scheme__ on a __mostly-diffusive__ transport: 

- Try a more advective setup (increasing Peclet number:
  `$Pe = \frac{|\mathbf{U}|L}{D_T}$`, where `L` is the domain's length)


## What now?

- Congratulations on completing this module; Its 
  [amazing assignment](https://classroom.github.com/a/sids7B2Z) is wainting
  for you!
- Next, we'll go into the details of each simulation-workflow step; and the
  first one is: __The Meshing step__
