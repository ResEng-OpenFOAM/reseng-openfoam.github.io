---
title: "06 - OpenFOAM Case Structure"
date: 2020-10-23
weight: 10
description: >
  Get familiarized with OpenFOAM case structure and file format.
mermaid: true
---

OpenFOAM solvers need to read a set of files to function properly. You can think
of these files as transplations between what you, the user, want and what the
solver expects. Check this diagram out:

{{<mermaid align="center">}}
graph TD;
    A[Solver start] -->|Runs until encounters| B(A dimensioned scalar called DT);
    B -->|Uses DT| C[Solver results]
    D(case file) -->|Specifying the value of| B
{{< /mermaid >}}


## OpenFOAM Case Structure

The idea here is that the solver knows about the existence of a variable (a word)
called DT (in its own "language") but it needs the user to translate it into the
user's language (Providing the actual value) throu a specific case file.

Hence, most of user interactions with OpenFOAM's solvers and utilities will
happen by editing text files prior to (or during) launching the application.

We call these files "dictionaries" and they are gathered into a single directory
, called a __case__.

These dictionaries have a common structure and can be recognized as OpenFOAM
files by looking into them. Take a look at the following diagram:

{{<mermaid align="center">}}
graph TD;
    A[fa:fa-folder Case] --> B[fa:fa-history Time Directories]
    A --> C[fa:fa-lock constant]
    A --> D[fa:fa-cogs system]
    B --> E(Usually starting with 0)
    C --> F(Time-independent stuff)
    D --> G(Simulation configs)

    E --- H("Initial and Boundary conditions <br>(p, U, T ...)")
    F --- I("Static mesh <br> Transport & Turbulence properties")
    G --- J("Discretization <br> Numerical Solver settings")

    style A fill:orange,stroke:red
    style B fill:orange,stroke:red
    style C fill:orange,stroke:red
    style D fill:orange,stroke:red
{{< /mermaid >}}

### Time directories:

Before we launch the simulation, we need to specify initial and boundary
conditions for key fields from our problem. Depending on the "time scheme"
we use, we may need multiple "levels" (at multiple time levels) of initial
values; Although, usually, we only need a single level, which we call "Time 0".

As the simulation progresses, OpenFOAM solvers will create new time directories
(One for each requested time level), which also can be used as initial values
for susequent simulations.

### Constant directory:

Everything that is time-independent, but relates to the physics problem at hand,
goes into the `constant` directory:

- The static mesh files (Or the initial state of the dynamic mesh).
- Model selection for problem features (Newtonian fluids, laminar flow, ... etc)

These features are not expected to change as the simulation progresses, hence,
it makes sense to store them only one in a central location.

### System directory:

That's where simulation settings are specified:

- Timestep control, Output settings
- Discretization schemes for various equation terms
- NS settings (tolerance ... etc)

## OpenFOAM File Format

Let's dig into the structure of the files contained in a typical OpenFOAM case.

### 1. The OpenFOAM file header

All OpenFOAM files (dictionaries) start with a sub-dictionary called the file
header:

```cpp
// File Header for the pressure object
FoamFile
{
  version 2;               // file format version, changed once in +20 yrs
  format  ascii;           // can also be binary
  class   volScalarField;  // What kind of object is this?
  object  p;               // Object name
}
```

I hope you can recognize that the previous snippet showed the file header for
the "pressure" field.

Here, the (sub)dictionary is named `FoamFile` and contains 4 __entries__.

> Note that dictionary contents are delimited by curly braces `{`, and `}`

Each entry is of the form:
```cpp
keyword    value;
//        |
//        |------ eveything from here to the last ";" is the value

// This is a comment by the way
```

More examples:

```cpp
   phases    (water oil gas);
//    |      |             |
//    |      |-------------|
//    |             |
// keyword        value (as a list of words)

   cells
   (
// ^
// |---- The Value for the cells keyword starts here
// |---- It's a list of "small" dictionaries
        labelToCell
        {
            value (0 1 2);
        }
        faceToCell
        {
            value (0 1 12);
        }
   );
// ^
// |---- End of the value of "cells" keyword
```

### 2. Detailed anatomy of the pressure file

The pressure field is involved in most of the simulations (or all of them), so,
let's take it as an example (File: `0/p`).

- First it starts with a "Banner". It serves no real purpose other than
  prettifying things a little.

```cpp
/*--------------------------------*- C++ -*---------------------------*\
| =========                 |                                          |
| \\      /  F ield         | foam-extend: Open Source CFD             |
|  \\    /   O peration     | Version:     4.0                         |
|   \\  /    A nd           | Web:         http://www.foam-extend.org  |
|    \\/     M anipulation  |                                          |
\*--------------------------------------------------------------------*/
```

- Next, comes the file header we talked about earlier.

- Then, there is a `dimensions` keyword which specifies the physical dimensions
  of the field. Its value is an ordered array of unit exponents:

```cpp
dimensions  [0    2      -2   0           0        0       0];
//          [Mass Length Time Temperature Quantity Current Luminous intensity];
```

> Note that these (Length squared, divided by time squared) are NOT the actual
> pressure dimensions. These are in fact the dimensions of __pressure divided
> by fluid density__. It's a convention in OpenFOAM's incompressible solvers.

- The following keyword is `internalField` which specifies the __list__
  of pressure values in mesh cells
  
```cpp
// Initialize p in all mesh cells to 0
internalField   uniform 0;
```

- The last keyword is called `boundaryField`, which defines boundary conditions
  for the pressure field:

```cpp
// boundaryField is subdictionary, no need to add at the end ";"
    boundaryField
    {
        movingWall              // Mesh boundary name
        {
            type zeroGradient;  // How should it be treated for this field
        }
        frontAndBack
        {
            type empty;
        }
    }
```

### More advanced file format features

#### Reusing values in a dictionary

One can define a custom keyword to hold the value of a certain variable:
```c
d           5;
// Here is how to use it
someKeyword $d;
```

#### Regular expressions in keywords

Keyword and dictionary names can be "(POSIX) regular expressions" instead of
literal strings. The following examples select multiple (matching) mesh
boundaries instead of a specific one:

```cpp
    boundaryField
    {
        // Zero Gradient for all boundaries ending with "Wall"
        ".*Wall"
        {
            type zeroGradient;
        }
        // Fixed Value for inlet and outlet (matching "inlet" OR "outlet")
        "(inlet|outlet)"
        {
            type fixedValue;
        }
    }
```

#### Include other dictionaries

The file format also allows for dictionay inclusion at any point:

```cpp
// The file is looked up in case directory
#include "customKeywords.cfg"
```

### Practice what you've learned about OpenFOAM's file format

> Please login to a shell on the `openfoam` container from your lab machine.
> This section will focus on hands-on discovering of case files.

### What's next?

It's recommended that you go through this lecture's 
[individual assignment](https://classroom.github.com/a/qu0WGf43).
Note that around 60% of "the learning" happens there, not while watching the
video!
