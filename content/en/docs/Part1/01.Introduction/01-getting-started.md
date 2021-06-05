---
title: "01 - Getting Started with OpenFOAM"
date: 2020-10-20
weight: 5
description: >
  Course objectives, prerequisites and help to setup the lab environment using LinuxOne Community machines.
math: true
mermaid: true
---

## Introduction to the software packages

### OpenFOAM

You can view OpenFOAM as

- a set of libraries and solvers for solving partial
differential equations,
- _numerically_, on a well defined "discret" domain
  - which we call the mesh
- using the Finite Volume Method (mostly)

Hence you can

- Make use of the __standard solvers__ directly if applicable to solve supported
  problems.
- But in the case of reservoir engineering problems, there is a limit to what
  can be done with the standard solvers

That's why taking the base code and developping our own solvers is such a big
catch.

These solvers typically run (from the command-line) on a set of files and
output simulation results as data files.

At this stage, OpenFOAM's role is (mostly) done; we only need to use a
Post-Processor to manipulate and visualize the generated data.

{{% alert title="Example" color="success" %}}
Solving a basic diffusive-problem involves solving an equation which looks like
this (In steady-state setups): 
`$$ \nabla \cdot (K\nabla T) = S$$`
Where:

- `$K$` is the diffusion coefficient
- `$T$` is the transported scalar field
- `$S$` is the source/sink term

There is a standard OpenFOAM solver called `laplacianFoam` which solves, well,
Laplacian equations. You can run the solver on a prepared set of files
(describing the diffusion problem)

```bash
(user@machine) laplacianFoam
```

And its log would look like (trimmed a little bit):

```
===================== BANNER =============================
Build  : 7-1ff648926f77
Exec   : laplacianFoam
Date   : Oct 19 2020
Time   : 15:37:05
Host   : "machine"
PID    : 22255
I/O    : uncollated
Case   : /tmp/flange
nProcs : 1
sigFpe : Enabling ...
fileModificationChecking : Monitoring run-time modified ...
allowSystemOperations : Allowing user-supplied system call
// * * * * * * * * * * * * * * * * * * * * * * * * * * * //
Create time
Create mesh for time = 1
SIMPLE: No convergence criteria found
Reading field T
Reading transportProperties
Reading diffusivity DT
----- Calculations for timesteps -----
END
```

{{% /alert %}}


### ParaView

ParaView is a common Post-Processing software used on a wide range of
data formats, including OpenFOAM's.

- Good Integration with OpenFOAM's solvers.
- Scriptable and can be driven by Python scripts.

Some of the most common data manipulation operations include:

- Common filters: Slices, Glyphs, Streams (to visualize flow path)
  and threshold-based extractions.
- ParaView also allows for Python-programmable filters.

{{% alert title="Example" color="success" %}}
Let's see one of the simplest ParaView's filters in action:

![ParaView Clip Filter](/course/part-1/img/01.Introduction/01-clip-filter.png)
{{% /alert %}}

### PyFoam

More advanced users could rely on the PyFoam library to

- Automate OpenFOAM tasks.
- Manage parallel jobs.
- Real-Time monitoing of simulation/solution properties.
- Easily perform parametrization and optimization studies.

PyFoam also provides facilities to

- Ease networking between OpenFOAM instances,
- Storing data in pickled format (and Pandas dataframes).

### cfMesh

Although standard meshing utilities (which come with OpenFOAM) are good enough
for most cases, there is a meshing utility called `cfMesh`, which

- Provides seamless tetrahedral, (2D-3D) hexahedral and polyhedral meshing
  workflows for __the lazy ones__ (me included).
- Eases complex mesh parametrization
- Only needs a geometry description for the domain and a maximal cell size!

## The Lab Environment

I hope you've already opened an account at
[linuxone.cloud.marist.edu](https://linuxone.cloud.marist.edu/)
so you can fire up a virtual machine on their infrastructure and use it as
a lab environment for your assignments and (open source) course projects.

This section will first describe your options, and then provide a setp-by-step
for getting your new virtual machine up and running.

### Option 01 : Work on your local machine

In fact, if you can, I encourage you to run on a Linux distribution locally
(Ubuntu might be a good starting point) where you'll install OpenFOAM from
official repositories.

In the early stages of this part of the course, you'll have to at least have
__ParaView installed on your local machine__.

On Linux, ParaView will get installed alongside OpenFOAM anyway.
On Other platforms, head to
[ParaView's Download page](https://www.paraview.org/download/)
and get the appropriate package

You want:

- The most up-to-date full package,
   - NOT the server-only version,
- Built with MPI and Python3 support.
   - On MacOS, it's OK to use the Python2-based package

To install OpenFOAM, follow the approprtiate (for your platform) instructions
from the [openfoam.org](https://openfoam.org/download/) website.

#### Advantages

- If you already have OpenFOAM installed, why bother using something else?

#### Difficulities

- If you're new to Linux, it might be challenging to keep up with the
  assignments. In that case, please use the in-cloud VM instead.

### Option 02 : Work on the cloud

If you're just starting up with Linux, connecting to a cloud machine and
going through the assignments and projects is probably more suited for you.

#### Advantages

- Local OS is irrelevant
- Faster setup, no need to manage a local Linux OS (e.g. if you're a Windows
  user).
- Practice what simulations in real cloud environments feel like.

#### Difficulties

- Becoming dependent on internet-connection
- Results visualization is still local

### Mix'n'Match

If you locally:

- Run a Docker-capable OS.
- Have enough CPU-RAM processing power.

You could build a Docker Swarm to emulate a network of OpenFOAM nodes for
parallel simulations. But this option is recommended only for advanced users
and we will not discuss it further.

### Setting up the remote server

The following diagram explains the whole workflow in the suggest lab
environment. You connect from your local machine to the remote one (over SSH)
which has an OpenFOAM docker image installed.

To use OpenFOAM, you just instantiate a container (based on the provided image)
and run the commands inside the container.

If things get messed up, that's fine; You just ditch the container and start
a fresh one!

{{<mermaid align="center">}}
graph TD;
    A[Local machine] -->|Connect To| B((LinuxOne machine));
    C(Docker Framework) --> |installed on| B;
    D(OF image) --- |Has Python3, OpenFOAM| C;
    D x--> |instantiates| E((Containers));

    click C "https://docker.com" "Official Docker website"
    style A fill:orange,stroke:red
    style B fill:cyan,stroke:cyan,draw:white
{{< /mermaid >}}

Let's now go through the steps to create a LinuxOne VM:

- We start at the homepage of your cloud account.
  Go `Service Catalog > Manage Instances` to get started

![](/course/part-1/img/01.Introduction/01-lab-setup/linuxone-community-01.jpg)

- Enter the following information

| Setting           | Value          |
|-------------------|-----------------|
| Type              | General purpose VM        |
| Name & Description| Choose a name for your VM    |
| Image (The OS)     | RHEL7.x |
| Flavor     | The one with 2 CPU cores |

- You also need to generate __and download__ an SSH key pair so you can
  access the machine later (obviously, choose better file names):

![](/course/part-1/img/01.Introduction/01-lab-setup/linuxone-community-06.jpg)
![](/course/part-1/img/01.Introduction/01-lab-setup/linuxone-community-07.jpg)

Make sure to __select__ the key pair you just created

![](/course/part-1/img/01.Introduction/01-lab-setup/linuxone-community-08.jpg)

- Hit the create button and wait for a couple of minutes, your machine will
  be ready in no time! (Note the IP address of your machine)

![](/course/part-1/img/01.Introduction/01-lab-setup/linuxone-community-10.jpg)

- Now, assuming you've downloaded the key file to your downloads folder, you
  need to run these commands on your local machine:
```bash
# This is a shell on your local machine, acting as regular user
# 1. Move key file to SSH configuration directory
(loc:~) mv ~/Downloads/remotesshkey.pem ~/.ssh/
# 2. Set correct permissions on the file (read-only to the file owner)
(loc:~) chmod 400 ~/.ssh/remotesshkey.pem
# 3. Start an SSH session to your remote machine:
#   - linux1: The default user on the remote machine
#   - xxx.xxx.xxx.xxx: Ip address of your remote machine
(loc:~) ssh -i ~/.ssh/remotesshkey.pem linux1@xxx.xxx.xxx.xxx
```

- Once logged in, on the remote machine, run:
```bash
# This is a shell on remote machine, acting as regular user (linux1)
# 1. Store the installation script URL as a variable
(rem:~) export GIST_URL="https://gist.githubusercontent.com/FoamScience/c5e1363cee5b0a402151045247eb4f9f/raw/create-s390x-openfoam-container.sh"
# 2. Download the script and run it
(rem:~) bash <(curl -s $GIST_URL)
# 3. Login to a new shell for things to work
(rem:~) sudo su
```
The script sets up Docker and download the necessary images for the OpenFOAM
containers. Its output should be similar to the following:
```
# Download and install Docker ...
# Add current user to docker's group ...
# Enable and start the docker service ...
# Spinup an OpenFOAM container ...
```
- You should then be able to run a bash shell from the container
  (named openfoam):
```bash
(rem:~) docker exec -it openfoam sudo su
# The previous command will open a shell on the container
# To close it, type "exit" (Or press Ctrl+D), and you're back
# to the remote machine
(con:~)$ exit
```


## Unit Assignment: Recon your remote machine

{{% pageinfo color="warning" %}}
You'll probably be asked to accept the invitation to join the 
[ResEng-OpenFOAM](https://github.com/orgs/ResEng-OpenFOAM) organization
on Github. That's the account I use to manage assignments/projects; so please
accept the invitation.
{{% /pageinfo %}}

Now, it's time for you to recon your new system!

You should take this opportunity to get familiarized with Github repositories,
and go through this unit's
__[individual assignment](https://classroom.github.com/a/7tvMm4nt)__. It was
designed so new comers can feel at home with a little bit of practice.

##### Should I shutdown the remote machine if I'm not working on it?

Yes, it is considered best practice. Why? Because when you go to sleep, a
person from the other half of the world may get up and fire up his own! No
need to strain the infrastructure for no good reason!

## Learn more

1. LinuxOne Community is hosted at
   [marist.edu](https://linuxone.cloud.marist.edu/cloud/), go there to create
   a free account.
2. I personally prefer to work on the command line, but many will find an SSH
   client, like [SnowFlake](https://github.com/subhra74/snowflake), useful.
3. The shell script we use to create the OpenFOAM container is 
   [hosted at Github](https://gist.githubusercontent.com/FoamScience/c5e1363cee5b0a402151045247eb4f9f/raw/create-s390x-openfoam-container.sh).
   It uses a Docker image I built specifically for the s390x architecture
   (x86-based ones will not work on LinuxOne machines).
5. Go grab the latest version of ParaView from 
   [its official website](https://www.paraview.org/download/) for your 
   (local) platform.

