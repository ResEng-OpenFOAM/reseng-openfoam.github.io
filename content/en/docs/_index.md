---
title: "Reservoir Engineering Simulations with OpenFOAM Course"
linkTitle: "Course Homepage"
permalink: "/course-homepage"
weight: 20
menu:
  main:
    weight: 20
---

{{% pageinfo %}}
The course is packaged in two **parts**:

- _**Part 1:**_ For those who never used OpenFOAM before, requires no (to
  little) programming skills
- _**Part 2:**_ For those who are already efficient in dealing with C++ code
  and have some experience with OpenFOAM cases (can simulate their own
  problems).

It's recommended that you open a 
[**IBM LinuxOne-Community**](https://linuxone.cloud.marist.edu/cloud/) Account before
beginning the course

Enjoy!
{{% /pageinfo %}}

{{% alert title="Licensing" %}}
This course is __not part__ of official OpenFOAM training.<br>
Other than using their software packages, this course has nothing to do
with OpenCFD, the producer and maintainer of OpenFOAM and its owner as a
trademark.

All packages explicitely mensioned in this course are licensed under some open
source license (Mostly Apache 2, MIT, or a compatible one).<br>
__OpenFOAM has a GPLv3 License__.
{{% /alert %}}

## What you'll learn

- [ ] 


## This is an Opinionated course!

Most of the **utilities** used throughout the course are of course replaceable;
I work with what's convenient for me - but it might not be the perfect choice
for you!

That said, there are some assumptions I make on your background:

- You are running a __Recent Linux Distribution__ on your local machine
  (Ubuntu-Like preferred). We'll carry out the simulations on cloud machines
  anyway, but you'll have to figure out the equivalent commands if you're 
  running something else on your local machine.
- You've read the first few (Five or so) sections of OpenFOAM's User Guide.
- You are willing to _write code_, _debug it_ and _share it_ with others.
- You have a Github account and know how to use Git for version control.

For Part 2 of this course, I assume:

- You do have some programming experience in some language
  (C++ and Python are preferred).

## Course workflow

1. The course is divided into two parts, each contains a set of modules.
2. Each module has a set of small units, individual and/or group assignments.
3. Individual assignments involve answering quiz-like questions and performing
   light tasks relating to OpenFOAM usage.
4. Group assignments are mini-projects to apply what you've learned in the whole
   module.
   - You can join a group of up to 3 individuals to carry these projects out.
5. At the end of each course part, going through a capstone project is 
   strongly suggested.
   - You can join a team of up to 5 individuals for this type of projects.
6. Some assignments will automatically open __pull requests__ for the instructor
   to provide feedback on your work.

It's recommended that you use the labs environment (we will help you set it up)
but it's completely optional.

## Rules for engaging in group activities

- We use __Github repos__ for the assignments: you can contribute to, review and improve the code 
  posted by your teammates.
- Joining existing groups should be preferred over creating new ones whenever
  possible.
- __Please don't meddle__ with the repos' history (no `squashes`, no `fixups`
  and public `rebase`) while working in a group!
  It breaks things for your teammates and makes them angry for no good reason.
- Use Pull Requests to merge branches (or at least, use the `--no-ff` option
  on the command line).
  It's important to keep track of who did what!
- Forking code posted by another group is in fact __encouraged__! But
  - You must be explicit about it. Use the "fork" button.
  - You should make significant improvements to their version.
  - Keep in mind that anyone can also fork your code too!

## Contributions

Spotted a **typo**? Think something may be a little "clearer" if said (written)
in a different way?
Just hit the `Edit this page` link on the top-right of this page,
make your changes, and commit them with a useful message.
I'll review your changes and accept them if feasible.

Do you think there is an issue in the course content you're viewing?
Hit the `Create course issue`
button on the top-right of this page to open a Github issue.
Explain what's the problem there.

At advanced stages of the course, you may be using some classes from my Open
Reservoir Simulation Research (OpenRSR) toolkit. If you notice an issue
(eg. Usage of outdated code in the course, or simply have a suggestion),
you can open a project issue by clicking on `Create a project issue` button.

Let's start learning, shall we?
