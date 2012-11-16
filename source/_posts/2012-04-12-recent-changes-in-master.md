---
layout: post
title: "Recent changes in master"
description: ""
category: updates
tags: [code, master, changes]
---

A couple of patches recently made their way into the master branch of CIlib which
affect the way CIlib is used. The changes were made to the stopping conditions,
topologies and control parameters.

### Stopping Conditions

In a bid to increase genericity (is that a word?) and remove duplicate code the
stopping conditions package was revamped. The new stopping condition classes are as
follows:

- MeasuredStoppingCondition which takes in a measurment an Objective and a target
The measurement is taken from the measurement package, objective can be either
Maximum or Minimum and target is the value the measurement must exceed (above
for Maximum, below for Minimum) for the algorithm to terminate.

- MaintatinedStoppingCondition whcih takes in another StoppingCondition and the
number of consecutive iterations for which the condition must hold before the
algorithm can terminate.

Here are a few examples showing the differences from the before and after the
changes.

#### Before
Java:

    StoppingCondition sc = new MaximumIterations(2000);

    StoppingCondition sc = new MinimumDiversity();
    sc.setMinimumDiversity(ConstantControlParameter.of(0.01));
    sc.setConsecutiveIterations(ConstantControlParameter.of(20));

XML:

    <addStoppingCondition class="stoppingcondition.MaximumIterations" maximumIterations="2000"/>

    <addStoppingCondition class="stoppingcondition.MinimumDiversity">
        <minimumDiversity class="controlparameter.ConstantControlParameter" parameter="0.01"/>
        <consecutiveIterations class="controlparameter.ConstantControlParameter" parameter="20"/>
    </addStoppingCondition>

#### After
Java:

    StoppingCondition sc = new MeasuredStoppingCondition(new Iterations(), new Maximum(), 2000);

    StoppingCondition sc = new MaintainedStoppingCondition(new MeasuredStoppingCondition(new Diversity(), new Minimum(), 0.01), 20);

XML:

    <addStoppingCondition class="stoppingcondition.MeasuredStoppingCondition" target="2000">
        <measurement class="measurement.generic.Iterations"/>
        <objective class="stoppingcondition.Maximum/>"
    </addStoppingCondition>

    <addStoppingCondition class="stoppingcondition.MaintainedStoppingCondition" consecutiveIterations="20">
        <condition class="stoppingcondition.MeasuredStoppingCondition" target="0.01">
            <measurement class="measurement.single.diversity.Diversity"/>
            <objective class="stoppingcondition.Minimum/>"
        </condition>
    </addStoppingCondition>


### Topologies

The main difference regarding topologies is how to get the best entity/entities.
The getBestEntity method has been removed from the Topology classes and has been
placed in the Topologies class as a static method along with a number of other
methods. This was done to prevent overcrowding the topology interface and to
separate concerns since a topology is only suppose to impose a structure on a
population of entities.

#### Before

    Particle currentBest = topology.getBestEntity();
    Particle gBest = topology.getBestEntity(new SocialBestFitnessComparator());

    Particle nBest = particle.getNeighbourhoodBest();
    // Although to the nBest could have changed so a more accurate way would be
    // to iterate through the neighbourhood of the particle and compare each one

#### After

    Particle currentBest = Topologies.getBestEntity(topology);
    Particle gBest = Topologies.getBestEntity(topology, new SocialBestFitnessComparator());

    Particle currentNBest = Topologies.getNeighbourhoodBest(topology, particle);
    Particle nBest = Topologies.getNeighbourhoodBest(topology, particle, new SocialBestFitnessComparator());


### Control Parameters

Before, using control parameters that could change (e.g. linear increasing/decreasing
parameters) meant that the user had to update the parameters every iteration.
This became a hassle if a control parameter was introduced into code that already
had a big inheritance structure since a mechanism for updating the control parameters
would have to be introduced into each of those classes.

The changes made allow a control parameter to automatically update its value when
the getParameter method is called. One implication of this is that parameters cant
be updated only once per iteration. This is a problem for algorithms that use multiple
stopping conditions or whose stopping conditions dont include maximum iterations
(ControlParameters updateon percentage completion of an algorith which is determined
by the stopping conditions). This problem is dealt with by introducing a new
ControlParameter: UpdateOnIterationControlParameter which takes in a delegate
control parameter and updates it at the end of each iteration.

Additionally, some parameters were merged to avoid duplicating code and control
parameters are only bounded if used with BoundedControlParameter (which is a class
now, not an interface).

#### Before
Java:

    // Only updated at the end of an iteration or when updateParameter was called
    LinearIncreasingControlParameter cp = new LinearIncreasingControlParameter();
    cp.setUpperBound(0.9);
    cp.setLowerBound(0.4);
    cp.setParameter(0.4);

    LinearDecreasingControlParameter cp = new LinearDecreasingControlParameter();
    cp.setUpperBound(0.9);
    cp.setLowerBound(0.4);
    cp.setParameter(0.9);

XML:

    <inertiaWeight class="controlparameter.LinearIncreasingControlParameter">
        <upperBound value="0.9"/>
        <lowerBound value="0.4"/>
        <parameter value="0.4"/>
    </inertiaWeight>

    <inertiaWeight class="controlparameter.LinearDecreasingControlParameter">
        <upperBound value="0.9"/>
        <lowerBound value="0.4"/>
        <parameter value="0.9"/>
    </inertiaWeight>

#### After
Java:

    // Updates whenever it is called
    LinearlyVaryingControlParameter cp = new LinearlyVaryingControlParameter();
    cp.setInitialValue(0.4);
    cp.setFinalValue(0.9);

    LinearlyVaryingControlParameter cp = new LinearlyVaryingControlParameter();
    cp.setInitialValue(0.9);
    cp.setFinalValue(0.4);

    // Will update otherCP at the end of an iteration
    UpdateOnIterationControlParameter cp = new UpdateOnIterationControlParameter();
    cp.setDelegate(otherCP);

    // Will always keep otherCP within bounds
    BoundedControlParameter cp = new BoundedControlParameter();
    cp.setBounds(new Bounds(-1, 1));
    cp.setDelegate(otherCP);

XML:

    <inertiaWeight class="controlparameter.LinearlyVaryingControlParameter" initialValue="0.4" finalValue="0.9"/>

    <inertiaWeight class="controlparameter.LinearlyVaryingControlParameter" initialValue="0.9" finalValue="0.4"/>

    <inertiaWeight class="controlparameter.UpdateOnIterationControlParameter">
        <delegate class="controlparameter.LinearlyVaryingControlParameter" initialValue="0.9" finalValue="0.4"/>
    </inertiaWeight>


If there are any questions join us on [irc chat](http://webchat.freenode.net/?channels=cilib).
