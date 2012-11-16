---
layout: post
title: "New packaging, usage and removal of simulator scripts"
description: "Removal of simulator scripts from the core distribution of CIlib. Functionality subsumed by simulator jar."
category:
tags: [sbt, simulator, executable]
---

In the past, CIlib was released as a distribution and included a few artifacts:

 - That actual CIlib "fat" jar file (together with external dependencies).
 - A collection of sample XML documents that defined the specifications for the XML based simulator.
 - Shell scripts for users to "run" the simulator, providing a XML specification.

We are continuously trying to improve CIlib and, as such, we have decided to remove the simulator
scripts in favor of letting the simulator provide the "running" functionality in the form of an executable jar
file.

Furthermore, from a source level, CIlib uses Maven as the project build tool but the fact that the project
wants to move to a Scala DSL replacement of the XML based simulator means that we should reevaluate the
use of Maven as well.

We will discuss each of these in the sections that follow.

### Maven, the unfriendly build tool

The changes mentioned above require us to rethink how we package CIlib. It was decided
to rather split out the "simulator project" from the library. The remaining library is
now focused on all the algorithms and is devoid of the simulator logic.

Going forward, maven will be dropped and will be replaced with
[SBT](http://www.scala-sbt.org). SBT is a much simpler, more powerful build system,
particularly since the way you configure SBT is using scala and the SBT DSL. This means that
much of the maven plugin XML verbosity is simply removed and replaced with a few lines
of code - an infinitely saner way of doing the build definition.


### Simulator scripts removed, yes that's right

Going forward, we will be providing a simple reference from which users can create
their own "simulator" scripts, but this script will not be provided as part of the
CIlib distribution.

The script is available from the gist below, and is essentially for reference
purposes only (however we encourage the submission of any "fixes" that you may have):

{% gist 3122738 %}

The gist provides a script that is friendly for Linux and Mac users - if any Windows
users require such a script, please either provide us with one or request that one is
created for you (not sure when we can get the time to figure that beast out though :P)

It is recommended that you place the script somewhere on your `$PATH`, so that it will be available
when needed. Please refer to the script itself for the specific variables that can be altered.

For any questions, please use the [mailing list](mailto:cilib-user@groups.google.com)
or contact one of us in the `#cilib` on Freenode.net

Just in case, the information above will also be available in the documentation,
available from the links above.
