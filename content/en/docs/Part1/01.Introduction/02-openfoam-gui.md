---
title: "02 - Is there a GUI for OpenFOAM?"
date: 2020-10-20
weight: 6
description: >
  Why the GUI isn't a big deal in OpenFOAM's context ...
---

{{< alert color="success" >}}
This is a popular question among new comers who want to start "clicking" on a
bunch of buttons to get their problems simulated.
{{< /alert >}}

The short answer is, there are _many_ (No official ones though), but the thing
with Graphical User Interfaces is that they stand in the way of "script-ability"
of the software, thus, making some tasks harder than they should be.

Convincing you, that GUIs are not (probably) the right way to approach OpenFOAM, is
not that hard!

Just try the following Linux commands yourself:

```bash
cd /tmp
mkdir imagesByMonths
cd imagesByMonth
mkdir 20{20..30}-{01..12}
ls
```

There, you just created 120 directories to organize your photos in a nice way.
I don't claim that GUIs are not capable of doing this! There are indeed some
file managers which offer such capabilities; but that's the problem! The GUI has
to offer the feature before you can use it.

GUI-less (if that's a word) software packages generally provide more freedom when
it comes to scriptability.

Here is the real thing: Sooner or later, you will need to run optimization
studies on your OpenFOAM cases (eg. for history matching, shape optimization ...
etc).  And because we don't depend on a specific GUI package, we can use any
optimization software we like. The only requirement is that the optimization
package itself must support black-box solvers (Virtually all of them do!).

> You'll be amazed how easy it is to generate very complex mesh structures
> using a macro language, m4 for example.

If you are still convinced that GUIs are the way to go, go ahead, choose one 
from this [wiki page](https://openfoamwiki.net/index.php/GUI). You'll throw it
away in a couple of days anyway.

On the other hand, having a post-processing package with no graphical interface
may be painful,
that's why ParaView has a nice GUI. Even so, we'll probably need to step over it
and write Python scripts to drive the software at some point!
