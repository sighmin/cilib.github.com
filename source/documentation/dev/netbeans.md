---
layout: docs
title: "Developing Cilib with Netbeans"
group: "dev-docs"
description: ""
---

This is a step-by-step guide to using Netbeans as your IDE for developing CIlib.
It is assumed that the getting started guide for either [Linux](getting-started.html) or [Windows](windows-configurations.html) was followed and is working.

### Step 1: Get the Netbeans Scala plugin

The nb-scala plugin is needed for Netbeans to work with Scala and SBT projects.
The following steps will install nb-scala:

- Download the correct version from [here](http://sourceforge.net/projects/erlybird/files/nb-scala/)
- Extract the compressed archive somewhere on your hard drive (remember the location)
- In Netbeans click on `Tools` -> `Plugins`
- At the top, click on the `Downloaded` tab
- Click the `Add Plugins...` button near the top
- Browse to the location (the one you had to remember) where the archive was extracted
- Select all the modules (Ctrl-click does the trick)
- Click the `Ok` button at the bottom
- Click the `Install` button near the bottom of the `Plugins` window
- Click `Next`, tick the `Accept` (only if you agree to the license) block then click `Update`
- If a validation warning appears it should be safe to click `Continue`
- At this point Netbeans will ask you to restart the IDE... please do so


### Step 2: Get the sbt-netbeans plugin

The sbt-netbeans plugin allows SBT projects wo be opened by Netbeans by generating
an Ant project using SBT commands. The following steps will install sbt-netbeans

- Open the global SBT plugins file usually situated at `~/.sbt/plugins/plugins.sbt` 
(`~` is your home folder). If it does not exist create it.
- Add the following to the end of the file
        resolvers += ScalaToolsSnapshots
        
        resolvers += "remeniuk repo" at "http://remeniuk.github.com/maven"
        
        libraryDependencies += "org.netbeans" %% "sbt-netbeans-plugin" % "0.1.4"
    Note the empty lines between each line of text.
- Save the file


### Step 3: Set up the project

The following should be done in the terminal:

- Change to the directory CIlib is in, e.g. if it is in `~/src/cilib` then
        $ cd ~/src/cilib
- Run SBT
        $ sbt
- Run the netbeans command
        > netbeans
- Exit SBT
        > exit


### Step 4: Open the project in Netbeans

CIlib should now be ready to be opened by Netbeans.

- Open Netbeans
- Click `File` -> `Open Project...`
- Browse to the location that the CIlib folder is in e.g. if the CIlib source code
is in `~/src/cilib` then browse to `~/src`. The CIlib folder should have an icon 
containing two red marks to the left of it.
- Select the CIlib folder and click `Open Project` near the bottom
- In the `Projects` view expand the cilib project and the `Libraries` folder
- Open the `cilib-library` and `simulator` sub-projects by double clicking on them

All the library code (algorithms, problems, etc) is in the `cilib-library` sub-project.


#### Done

Please report any errors found either on the CIlib github page or on IRC. If 
there are people in the channel but are not responding leave a message anyway,
someone will eventually see it.


