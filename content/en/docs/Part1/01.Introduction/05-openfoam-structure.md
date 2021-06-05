---
title: "05 - OpenFOAM Structure"
date: 2020-10-22
weight: 9
description: >
  Get familiarized with the most important OpenFOAM installation files and
  directories.
mermaid: true
---

The Docker container we created and got familiarized with in the first unit
of this module had a recent OpenFOAM version already installed.

Now it's time to dig into OpenFOAM installation files.


## OpenFOAM Installation directory

Depending on what **users** you want to have access to OpenFOAM, one can
perform:

- A System-wide install: where all users have access to OpenFOAM's solvers and
  utilities. This is done by installing to a system location
  (eg. `/opt/openfoam8` for OpenFOAM v8)
  - This is the approach I used to install OpenFOAM in the container (See 
  `/opt/OpenFOAM-dev`)

- A User-specific install: where only one user will have access to OpenFOAM
  binaries. Usually, the user will winstall it under his home directory
  (`$HOME/OpenFOAM`)

To discover where OpenFOAM is installed on your system:
```bash
(of@con:run) echo $FOAM_INST_DIR
```

{{% alert title="Example" color="success" %}}
If we were to install OpenFOAM v8 as the user `of`, the installation files would
go to `/home/of/OpenFOAM/OpenFOAM-8`.

And the `FOAM_INST_DIR` environment variable will say: `/home/of/OpenFOAM`.

This way, we can also install additional versions (and forks) if we need to.
{{% /alert %}}

No matter how the software installed, if a users use it, they will have
some files in their `$HOME/OpenFOAM/$USER-<VERSION>`; where `<VERSION>` is
the version number of the software.

> Because the container has the latest(-ish) development version of OpenFOAM,
> the "version number" will actually say `dev`, so the mentioned directory
> would expand to `/home/of/OpenFOAM/of-dev`

## OpenFOAM directories

The installation comes with the following directories (the diagram 
attempts to give a brief "usage" summary for each directory; you'll pick it up
as you go):

{{<mermaid align="center">}}
graph LR;
    A["Parent OpenFOAM Dir."];
    style A fill:orange,stroke:red

    A --> B(fa:fa-file-code applications);
    A --> F(fa:fa-code src);
    A --> C(fa:fa-file-excel bin);
    A --> D(fa:fa-cog etc);
    A --> E(fa:fa-book-reader doc);
    A --> G(fa:fa-database tutorials);


    subgraph "Source Code"
    B --- H("Source Code for <br> solvers & utils")
    H --- I("Usage: Figure out how solvers work,<br>solved equations ... etc")

    F --- P("Library source code")
    P --- Q("Usage: Debug cases and<br>find out where things are failing")
    end

    C --- J("System executables")
    J --- K("Usage: control jobs,<br>Test installation .. etc")

    D --- L("OpenFOAM configuration")
    L --- M("Usage: Configure OpenFOAM settings,<br>sample case dictionaries")

    subgraph "User Docs"
    E --- N("Code documentation")
    N --- O("Usage: C++ code docs,<br>The user guide")

    G --- R("Tutorial cases")
    R --- S("Usage: Solver verification,<br>Copy-pasting from similar cases,<br>Learning ... etc")
    end

    style I fill:lightgreen,stroke:lightgreen
    style Q fill:lightgreen,stroke:lightgreen
    style K fill:lightgreen,stroke:lightgreen
    style M fill:lightgreen,stroke:lightgreen
    style O fill:lightgreen,stroke:lightgreen
    style S fill:lightgreen,stroke:lightgreen

{{< /mermaid >}}

{{% alert title="Example" color="success" %}}
If we were to install OpenFOAM v8 as the user `of`, the installation files would
go to `/home/of/OpenFOAM/OpenFOAM-8`.

And the `FOAM_INST_DIR` environment variable will say: `/home/of/OpenFOAM`.

This way, we can also install additional versions (and forks) if we need to.
{{% /alert %}}
## The tutorials directory

The tutorials directory is extremely useful when:

- You want to start new cases.
  - Just copy over the most similar (to your problem) tutorial case and start
    adjusting ...
- You're confused about a keyword.
  - Eg. Want to find out what NS is preferred for a specific solver? Just
    cross-search tutorial dictionaries for matches (You'll learn how to that
    later)
- You want to find out which solvers may be suited for your problem
  - A tutorial case may be compatible with many solvers;
  - Find the closest case to your problem, and you'll find the appropriate
    solvers you need to run.


### Now what?

This unit is not associated with an assignment; you can continue 
to the next one immediately.
