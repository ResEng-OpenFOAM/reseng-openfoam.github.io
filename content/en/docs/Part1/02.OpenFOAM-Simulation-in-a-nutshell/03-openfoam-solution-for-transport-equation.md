---
title: "03 - OpenFOAM Solution for the transport problem"
date: 2020-10-29
weight: 7
description: >
  Build OpenFOAM cases to reflect the transport problem.
math: true
---

In this unit, we'll, again, solve the same example problem; But this time, we
are using OpenFOAM (Of course, this unit will be consistent with the previous
one). The basic workflow steps stay exactly the same.

> It's recommended that you __actualy__ follow along using the lab machine.
> So, create a case (named `intro` for example) in your run directory 
> (`$FOAM_TUTORIALS/basic/scalarTransportFoam/pitzDaily` is a good starting
> point).

## Mesh generation: Introducing `blockMesh`

The objective of this section is to generate an OpenFOAM mesh, matching the
following design:

![OpenFOAM transport 1D mesh](/course/part-1/img/02.OpenFOAM-Simulation-in-a-nutshell/02-mesh-surface-normals.png)

To do so, we'll use a standard utility, called `blockMesh`, which is basically
a __multi-block__ mesh generator, controlled by a meshing dictionary (a file
in the case's directory describing mesh blocks).

In recent OpenFOAM versions, this dictionary is located at
`system/blockMeshDict`

- But it used to be located at `constant/polyMesh/blockMeshDict`, which still
  works.

> A detailed discussion of this file's contents will be provided later, for now
> just __nod along__ and

1. The `pitzDaily` tutorial case is the closest one to our problem, but its
   mesh is a bit complicated for our purpose, thus, one can grab the mesh from
   another case:
   ```bash
   (of@con:intro) cp $FOAM_TUTORIALS/incompressible/icoFoam/cavity/cavity/system/blockMeshDict system/
   ```
2. This dictionary usually starts off with a list of block vertices, where each
   mesh block point is labeled in the order it appears in:
   ```cpp
vertices
(
        ( 0   0   0   )  // vertex 0
        ( 0.9 0   0   )  // vertex 1
        ( 0.9 0.1 0   )     
        ( 0   0.1 0   )
        ( 0   0   0.1 )
        ( 0.9 0   0.1 )
        ( 0.9 0.1 0.1 )
        ( 0   0.1 0.1 )
);
   ```
3. Then, a set of hexagonal blocks is specified (in this case, just one,
   connects points 0 through 7 and split into 9 cells in the x-direction)
   ```cpp
blocks
(
        hex (0 1 2 3 4 5 6 7) (9 1 1) simpleGrading (1 1 1)
);
   ```
   Note that there is a single cell in the y and z directions.
   Also, the `simpleGrading (1 1 1)` bit specifies a uniform cell size in all
   three directions.
4. We also have the option to specfy any __curved__ block edges in the `edges`
   list, but we'll leave that alone for now:
   ```cpp
edges
(
);
   ```
5. It only remains to specify boundary patches and their types. For our case,
   we will have the far left face (`$f_0$`) constinute the `inlet` boundary
   patch. Its type should generic (`patch`), and we specify the face as a list
   of its vertices' labels.
   ```cpp
boundary
(
        inlet                     // Patch name
        {
          type patch ;            // Generic patch type
          faces ( ( 0 4 7 3 ) ) ; // Inlet has one face
        }
        outlet
        {
          type patch ;
          faces ( ( 2 6 5 1 ) ) ;
        }
        noFlow
        {
          type empty ;             // No flow in perpendicular direc. 
          faces ( ( 3 7 6 2 ) ( 1 5 4 0 ) ( 0 3 2 1 ) ( 4 5 6 7 ) ) ;
        }
);
   ```
   The outlet is treated similarly, taking the face on the other end of the
   mesh (`$f_9$`). The remaining faces are collected into a `noFlow` patch
   with an `empty` type.

> The `empty` patch type is a way of saying: Ignore these faces when summing
> integrals cell faces (Revisit the previous unit). In this case, It's a simple
> way to say that the flow is unidirectional.

That's all, all we need to do is to run the meshing command in the case's
directory:
```bash
(of@con:intro) blockMesh
```

To visualize the mesh, one should move the initial directory away because it's
not (yet) compatible with the generated mesh:
```bash
(of@con:intro) blockMesh
(of@con:intro) touch intro.foam
(of@con:intro) paraview intro.foam
```

![OpenFOAM 1D mesh](/course/part-1/img/02.OpenFOAM-Simulation-in-a-nutshell/03-openfoam-mesh.png)

{{% alert title="Lost your way?" color="warning" %}}

There is a [Github repo](https://github.com/ResEng-OpenFOAM/res-eng-openfoam-introduction)
holding case files at each stage of the simulation. The
git tag for this stage is __meshingStage__.

You can checkout the tag if things are not working correctly for you; or better,
compare (`git diff`) your work with the contents of the tag.

{{% /alert %}}

## Solver and governing equations

The next step is to pick the corresponding solver for our flow equation which is
`scalaTransportFoam`, where the solved governing equation has three terms:

```cpp
solve
(
    fvm::ddt(T)                 // dT/dt, null in our stead_state case
    + fvm::div(phi, T)          // divergence(U T)
    - fvm::laplacian(DT, T)     // - divergence(D_T grad(T))
);
```

Where:

- `phi` is the phase's flux
- `U` is the flow's velocity
- `$D_T = \frac{K}{\rho}$` is the diffusion coefficient

The transported variable is named `T` in this case (as opposed to `$\phi$` in 
the previous units) and the equation is practically the one from the manual
approach, divided by the fluid's density `$\rho$`.

> For a quick description of the solver, look for "Description" in 
> `$FOAM_SOLVERS/basic/scalarTransportFoam/scalarTransportFoam.C`

## Initial and boundary conditions

After we decide on a solver to use, It's wise to setup initial and boundary
conditions for our fields (`T`, and `U`).

To be fully consistent with the previous unit, set:

- `$\mathbf{U} = \begin{bmatrix} 0.03 \\ 0 \\ 0 \end{bmatrix}$`
- `$D_T = 0.01$`

> Move `0.orig` back to `0` with: `(of@con:intro) mv 0.orig 0`

Back to our case files, The `T` field should have temperature dimensions
(because that's what the solver expects), and
it should reflect the boundary conditions from the previous unit (fixed values:
1 at the inlet, 0 at the outlet).

Initial values for the `T` field will also take zeroed values to compare the
numerical method with the manual approach.

```cpp
dimensions      [0 0 0 1 0 0 0];

internalField   uniform 0;

boundaryField
{
    inlet
    {
        type            fixedValue;
        value           uniform 1;
    }
    outlet
    {
        type            fixedValue;
        value           uniform 0;
    }
    noFlow
    {
        type            empty;
    }
}
```

For the velocity, `zeroGradient` boundary condition type should be applied to
both the inlet and the outlet of the domain:

```cpp
dimensions      [0 1 -1 0 0 0 0];

internalField   uniform (0.03 0 0);

boundaryField
{
    inlet
    {
        type            zeroGradient;
    }
    outlet
    {
        type            zeroGradient;
    }
    noFlow
    {
        type            noSlip;
    }
}
```

> By default, `constant/transportProperties` dictionary specifies `DT = 0.01`
> (and we're fine with it)

{{% alert title="Lost your way?" color="warning" %}}
Again, the 
[Github repo](https://github.com/ResEng-OpenFOAM/res-eng-openfoam-introduction)
has all case files up until this point. The git tag for this stage is
__ICsBCsConfigured__.
{{% /alert %}}

## Numerical schemes and solution settings 

As for the numerical schemes used, let's use the same ones from the manual
approach:

```cpp
// File: system/fvSchemes

ddtSchemes
{
    // Default Time-derivative approximation method
    default     steadyState;
    // default means all 'ddt' terms will be approximated this way
    // In our case, that's the 'fvm::ddt(T)' term
}

gradSchemes
{
    // Whenever there is a need to calculate the gradient of a field, use:
    // Gaussian integrals evaluated using a linear profile
    default     Gauss linear
}

divSchemes
{
    // It's bad practice to set a default option for divergence schemes
    default     none;

    // Then, we must specify a scheme for each div term
    // linear is NOT a good choice in most cases, don't make it a habit
    div(phi,T)  Gauss linear;
}

laplacianSchemes
{
    // It's not that bad set a default option for divergence schemes, but
    default     none;

    // Then, we must specify a scheme for each laplacian term
    laplacian(DT,T)  Gauss linear uncorrected;
}

interpolationSchemes
{
    // What to do when there is a need to interpolate?
    default     linear;
}

```

You'll have to nod along and accept `Gauss linear uncorrected` as the diffusion
scheme for now. Let's just say the mesh quality has a considerable impact on the
`laplacian` 's accuracy, thus we might need to correct things in some cases, but
not in this simple one!!

Also, let's no consern ourselves with `snGradSchemes` (schemes for 
surface-normal gradients) for now, so, just leave them as they are.

After using these schemes to derive the matrix to solve, OpenFOAM will need to
actually solve it:

```cpp
// File: system/fvSolution

// A dictionary for linear solvers
solvers
{
    // Solvers for T's equation (our only equation)
    T
    {
        // The linear solver for T
        // In OpenFOAM, Gauss-Seidel is not a fully fledged solver,
        // instead, it's just a method using for matrix smoothing,
        // thus, we say to use the smoother as the solver here:
        solver  smoothSolver;

        // Then specify the smoother as Gauss-Seidel here:
        smoother GaussSeidel;

        // To compare results with the manual approach,
        //let's disable any native convergence control:
        tolerance   1e-12;
        relTol      0;

        // But let's force the linear solve to perform ONE linear iteration
        // per timeStep
        maxIter     1;
    }
}

SIMPLE
{
    // Again, we don't need any corrections
    nNonOrthogonalCorrectors 0;
}
```

By setting `tolerance 1e-12;` (required difference between equation residuals
at consecutive time levels), and `relTol 0;` (required relative reduction in
residuals)
, we completely throw out the notion of "convergence" (assuming a tolerance of
`1e-12` is hard to achieve). The simulation will 
(probably) never converge with these settings, but that's what we want because
we want to __run for a specified number of iterations__ (37 to be exact,
deduced from the manual approach).

Now, the notion of a __TimeStep__ also doesn't quite hold itself in a
steady-state simulation, does it? In this case, by a TimeStep, we mean a
"solver iteration", but not a "linear solver iteration": One Timestep may need
multiple linear solver iterations to converge. That's also why we set `maxIter
1;` to bring the two notions together.

{{% alert title="Lost your way?" color="warning" %}}
The git tag for this stage is __NSAndSolConfigured__. Check it out in this same
[Github repo](https://github.com/ResEng-OpenFOAM/res-eng-openfoam-introduction)
{{% /alert %}}

## Running the simulation

We only need to tweak `system/controlDict` so that it reflects what we did back
in the manual approach (previous unit):

```cpp
// File: system/controldict

application     scalarTransportFoam;

// ALWAYS start from startTime below, do not continue previous simulations
startFrom       startTime;
startTime       0;

// STOP when you reach time = endTime
stopAt          endTime;
endTime         40;  // This should be in time units (seconds) but
                     // as this is steady-state, it's the number of iterations

// IRRELEVANT in our case, but
deltaT          1;   // Set it so that endTime is valid

// WHEN and HOW to write solution to disk
writeControl    timeStep;  // Output solution every n*timeStep
writeInterval   1;         // Specifies the "n" in previous comment
writePrecision  8;         // Writing precision of floating points
```

`deltaT` is obviously irrelevant in our case: Your results should not differ
if you use any other value for this keyword, however you can see that
`writeInterval` would have a different effect.

The remaining settings are less relevant, and we can ignore them for the time
being. So, let's start the simulation:

```bash
# Create the mesh and check its quality
(of@con:intro) blockMesh && checkMesh
# Run the solver and keep a log file
(of@con:intro) scalarTransportFoam | tee log.scalarTransportFoam
```

{{% alert title="Lost your way?" color="warning" %}}
There is also a git tag for this stage (__RunReady__) in the case's
[Github repo](https://github.com/ResEng-OpenFOAM/res-eng-openfoam-introduction)
{{% /alert %}}

### Investigating results

Here is a pretty table of `T` values at each iteration:

| Iteration | Cell 0 | Cell 1 | Cell 2 | Cell 3 | Cell 4 | Cell 5 | Cell 6 | Cell 7 | Cell 8 |
|-----------|--------|--------|--------|--------|--------|--------|--------|--------|--------|
| 00 |        0 |       0 |       0 |     0   |    0    |       0 |      0 |       0 |     0 |
| 01 | 0.7301 | 0.4198 | 0.2414 | 0.1388 | 0.0798 | 0.0458 | 0.0263 | 0.0151 | 0.0061 |
| 02 | 0.8434 | 0.5875 | 0.3968 | 0.2621 | 0.1702 | 0.1090 | 0.0691 | 0.0423 | 0.0171 |
| 03 | 0.8887 | 0.6796 | 0.5022 | 0.3611 | 0.2540 | 0.1754 | 0.1188 | 0.0756 | 0.0305 |
| 04 | 0.9135 | 0.7387 | 0.5782 | 0.4404 | 0.3278 | 0.2390 | 0.1695 | 0.1104 | 0.0445 |
| 05 | 0.9295 | 0.7802 | 0.6358 | 0.5049 | 0.3919 | 0.2974 | 0.2179 | 0.1442 | 0.0582 |
| ... | ... | ... | ... | ... | ... | ... | ... | ... | ... | 
| 35 | 0.9888 | 0.9588 | 0.9188 | 0.8653 | 0.7938 | 0.6978 | 0.5687 | 0.3946 | 0.1592 |
| 36 | 0.9888 | 0.9591 | 0.9192 | 0.8659 | 0.7945 | 0.6985 | 0.5694 | 0.3950 | 0.1594 |
| 37 | 0.9889 | 0.9593 | 0.9196 | 0.8664 | 0.7951 | 0.6991 | 0.5699 | 0.3954 | 0.1595 |


But don't just take my word for it; you can produce such tables with a single
bash command (By looking for "nonuniform" into the `T` file for all iterations):

```bash
for i in {1..37}
    do grep nonuniform $i/T | sed "s/.*(\(.*\)).*/$i\t\1/g"
done
```

The `internalField` of `37/T` for example would look like this:
```
internalField   nonuniform List<scalar> 9(0.98896731 ...);
```
so we just take what's inside the round brackets (as a bonus, the `sed` commands
adds the iteration number at the start of each line (`$i`)).

## What now?

- You should proceed to the next unit to learn how to properly visualize
  simulation results (Presenting data in tables is not catchy after all).
