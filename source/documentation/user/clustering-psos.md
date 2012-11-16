---
layout: docs
title: "Clustering PSO Algorithms"
description: ""
group: "user-docs"
tags: [code, master, changes, clustering, pso]
---

A number of Clustering PSO algorithms were added to CILib. These clustering PSOs 
include a number of variations to the standard clustering PSO described by van 
der Merwe 2003. Note that the entity used is called a ClusterParticle and this
entity holds a set of centroids, so the "dimension" of the particle refers to the 
position of a centroid.

# Available Algorithm Configurations

- Standard Clustering PSO: This is a PSO where particles represent sets of cluster 
centroids. Fitness is measured using the quantization error, which measures the 
average distance of data patterns to the centroid to which they belong. Each particle
 represents set of possible centroid positions and the algorithm attempts to minimize
 the distance between data patterns belonging to their relevant centroid.
		
- Reinitializing Clustering PSO: This is the standard PSO adapted to perform better in
a dynamic environment. As a change occurs, all or a portion of the total particles in
the population are reinitialized. 

- A Multi-Swarm Data Clustering PSO: This algorithm is especially designed for dynamic
environments. It involves initializing a number of swarms independent from each other.
If two swarms are have a global bests which are located near each other, then one of 
the swarms is re-initialized. This is done in order to avoid convergence around local 
minima or old values of the global best. If all three swarms converge, the one with the 
worst fitness is re-initialized in order to inject diversity. It is a set of Clustering 
PSOs running in parallel and attempting to optimize different parts of the search space. 
The global best value of the best fit swarm is the one that will be chosen as the 
algorithm's global best.

- Co-operative PSO: This algorithm involves optimizing each dimension of a problem separately
by creating n 1-dimensional swarms (where n is the dimensionality of the problem). A context
particle is used in order to calculate fitness (which is dependant on all dimensions). This 
particle consists of the best solution found for each dimension of the problem, and fitness of
a particle is calculated by replacing the dimension being dealt with in the context particle 
and calculating the fitness.

- Co-operative Multi-swarm PSO: This involves a cooperative PSO where each dimension of the 
Cluster Particle, i.e. each centroid, is a separate swarm. Each swarm is optimized using the 
multi-swarm algorithm and fitness is calculated using a context particle that contains the best
positions of each swarm.

# Implementation Details

## Standard Data Clustering PSO
- ClusterCentroid: a Type that represents a centroid. What makes it different from a vector is 
the fact that it holds a set of data patterns which are assigned to the cluster during the 
algorithms execution.
		
- CentroidHolder: a Type which represents a two dimensional vector. It holds ClusterCentroids.

- ClusterParticle: a form of Particle which handles a candidate solution of type CentroidHolder 
correctly.

- CentroidInitializationStrategy: a wrapper initialization strategy, which initializes the particle 
appropriately using the delegate InitializationStrategy.  If the delegate is chosen to be 
RandomBoundedInitializationStrategy, sets the bounds for each dimension to the bouds of the dataset.

- DataPatternInitializationStrategy: an initialization strategy which initializes particles to 
positions of randomly selected patterns.

- QuantizationErrorMinimizationProblem: the Quantization Error problem, where the result for a fitness
calculation is determined.

- CentroidBoundaryConstraint: a BoundaryConstraint wrapped which handles CentroidHolders appropriately.

- DataClusteringPSO: The data clustering algorithm

- SlidingWindow: This class determines the portion of the dataset that needs to be used by the 
algorithm at any point in time.

- DataDependantPopulationInitializationStrategy: This is a wrapper initialization strategy designed to
handle the initialization of ClusterParticles. Its main purpose is gathering information about the 
dataset required by the entity initialization strategy.

- StandardDataClusteringIterationStrategy: The iteration strategy of the standard clustering PSO.

## Re-initializing Data Clustering PSO

- ReinitializingDataClusteringIterationStrategy: The iteration strategy which re-initializes 
particles after a change has occurred. 
		
- SinglePopulationDataClusteringAlgorithm: The parent class of the 
ReinitializingDataClusteringIterationStrategy and the StandardDataClusteringIterationStrategy

## Multi-swarm Data Clustering PSO

- ClusteringMultiSwarmIterationStrategy: The multi-swarm iteration strategy designed to handle
a clustering problem where the solution is a CentroidHolder, not a Vector. 

## Cooperative Data Clustering PSO

- CooperativePSO: the cooperative PSO algorithm
		
- CooperativeDataClusteringPSOIterationStrategy: The standard cooperative PSO iteration strategy

- DynamicCooperativeDataClusteringPSOIterationStrategy: This class was added for experimentation.
It is a wrapper to the CooperativeDataClusteringPSOIterationStrategy which re-initializes the 
population once a change is detected. This did not work well at all. 


## Cooperative Multi-swarm Data Clustering PSO

- AbstractCooperativeIterationStrategy: This class was created as a parent to hold the co-operative
 PSO functionality that both, the Co-operative Data Clustering algorithm and the Cooperative 
Multi-swarm algorithm both needed.
			
- CooperativeMultiSwarmIterationStrategy: This class separates the ClusterParticles and creates the
swarms that the multi-swarm algorithm them optimises. Note that the number of swarms is the number 
of dimensions +1 as the weakest swarm re-initializes once all swarms have converged. This extra swarm 
is called the explorer swarm as it explores the environment.

## Other changes
- ClusterCentroids: This is a measurement which outputs all the centroids for each iteration and for 
each simulation to the final text file. Note CILib allows for one to output vectors but CIDA does not 
know how to handle them. I have created my own program to average the centroid positions and the 
fitness values and output them to a file.
			
- Validity Indexes:

    1. Validity Index: Abstract class for validity indexes    
    2. David Bouldin Validity Index
    3. Dunn's Validity Index
    4. Halkidi Vazirgiannis Validity Index
    5. Ray Turi Validity Index
    6. Favouring Ray Turi Validity Index


- RandomBoundedInitializationStrategy: modified by adding a method to change the lower and upper 
bounds according to the dimension being initialized was added. A method to set the array of bounds 
was implemented. 


# XML Examples

## Standard Data Clustering PSO

The dataset is provided to the algorithm as the sourceURL. A sliding window is created (there 
is a default sliding window so it is not necessary). The StandardDataClusteringIterationStrategy is used
for a standard data clustering PSO. If a boundary constraint is required, it is added as the delegate of 
CentroidBoundaryConstraint. The initialization strategy must be the DataDependantPopulationInitializationStrategy
as ClusterParticles need to be initialized and the dataset must be sent to the initialization strategy.
The centroidInitialisationStrategy can be StandardCentroidInitializationStrategy or DataPatternInitializationStrategy.
The positionInitialisationStrategy then becomes the delegate of the StandardCentroidInitializationStrategy. The
velocityInitialisationStrategy and personalBestInitialisationStrategy become delegates of their appropriate 
centroidInitializationStrategies.

XML:

    <algorithm id="clusteringPSO" class="clustering.DataClusteringPSO" sourceURL="src/test/resources/datasets/clusters.arff">
        <addStoppingCondition class="stoppingcondition.MeasuredStoppingCondition" target="6000"/>
        <window class="clustering.SlidingWindow"/>
        <iterationStrategy class="clustering.iterationstrategies.StandardDataClusteringIterationStrategy">
            <boundaryConstraint class="problem.boundaryconstraint.CentroidBoundaryConstraint">
                <delegate class="problem.boundaryconstraint.ClampingBoundaryConstraint"/>
            </boundaryConstraint>
        </iterationStrategy>
        <initialisationStrategy class="algorithm.initialisation.DataDependantPopulationInitializationStrategy">
            <delegate class="algorithm.initialisation.ClonedPopulationInitialisationStrategy"/>
            <entityNumber value="50"/>
            <entityType class="clustering.entity.ClusterParticle">
                <positionInitialisationStrategy class="entity.initialization.RandomBoundedInitializationStrategy"/>
                <centroidInitialisationStrategy class="entity.initialization.StandardCentroidInitializationStrategy"/>
            </entityType>
        </initialisationStrategy>
    </algorithm>

## Reinitializing Data Clustering PSO
Here there are two changes. One is the iteration strategy being used: ReinitializingDataClusteringIterationStrategy.
The second difference is seen by the sliding window. Here, because we are now using a dynamic problem, a frequency 
of 2000 is set when the total iterations are 6000. This means that the window will in the end have been over 3 
different time-steps. The window size is 30 because the dataset is of size 90. So during the first 2000 iterations
30 of the data patterns will be used, at the 2000th iteration, the window slides to the next 30.

XML:

    <algorithm id="clusteringPSO" class="clustering.DataClusteringPSO" sourceURL="src/test/resources/datasets/clusters.arff">
        <addStoppingCondition class="stoppingcondition.MeasuredStoppingCondition" target="6000"/>
        <window class="clustering.SlidingWindow" windowSize="30" frequency="2000"/>
        <iterationStrategy class="clustering.iterationstrategies.ReinitializingDataClusteringIterationStrategy">
            <boundaryConstraint class="problem.boundaryconstraint.CentroidBoundaryConstraint">
                <delegate class="problem.boundaryconstraint.ClampingBoundaryConstraint"/>
            </boundaryConstraint>
        </iterationStrategy>
        <initialisationStrategy class="algorithm.initialisation.DataDependantPopulationInitializationStrategy">
            <delegate class="algorithm.initialisation.ClonedPopulationInitialisationStrategy"/>
            <entityNumber value="50"/>
            <entityType class="clustering.entity.ClusterParticle">
                <positionInitialisationStrategy class="entity.initialization.RandomBoundedInitializationStrategy"/>
                <centroidInitialisationStrategy class="entity.initialization.StandardCentroidInitializationStrategy"/>
            </entityType>
        </initialisationStrategy>
    </algorithm>

## Cooperative Data Clustering PSO
Use the same xml shown above to create a Standard Data Clustering PSO. Then add this PSO to the cooperative PSO n times, 
where n is the number of clusters required (i.e. the dimensions).  An example is shown below for 3 clusters:

XML:
    
    <algorithm id="multiPopulationPSO" class="clustering.CooperativePSO">
        <iterationStrategy class="clustering.iterationstrategies.CooperativeDataClusteringPSOIterationStrategy"/>
        <addStoppingCondition class="stoppingcondition.MeasuredStoppingCondition" target="6000"/>
        <addPopulationBasedAlgorithm idref="clusteringPSO"/>
        <addPopulationBasedAlgorithm idref="clusteringPSO"/>
        <addPopulationBasedAlgorithm idref="clusteringPSO"/>
    </algorithm>

## Cooperative Context Re-initializing Data Clustering PSO
This is the same as the cooperative pso, but the re-initializing version. The standard data clustering 
algorithm is setup normally, but it's sliding window is set to slide by adding a frequency and window size.

XML:

    <algorithm id="multiPopulationPSO" class="clustering.CooperativePSO">
        <iterationStrategy class="clustering.iterationstrategies.DynamicCooperativeDataClusteringPSOIterationStrategy"/>
        <addStoppingCondition class="stoppingcondition.MeasuredStoppingCondition" target="6000"/>
        <addPopulationBasedAlgorithm idref="clusteringPSO"/>
        <addPopulationBasedAlgorithm idref="clusteringPSO"/>
        <addPopulationBasedAlgorithm idref="clusteringPSO"/>
    </algorithm>


## Multi-swarm Data Clustering PSO
Create a normal data clustering PSO as described above. Set the window size and frequency. Add n of these standard
clustering PSOs to the muli-population algorithm, where n is the number of clusters required.

XML:

    <algorithm id="multiPopulationPSO" class="pso.multiswarm.MultiSwarm">
        <multiSwarmIterationStrategy class="pso.multiswarm.StandardClusteringMultiSwarmIterationStrategy"/>
        <addStoppingCondition class="stoppingcondition.MeasuredStoppingCondition" target="6000"/>
        <addPopulationBasedAlgorithm idref="clusteringPSO"/>
        <addPopulationBasedAlgorithm idref="clusteringPSO"/>
        <addPopulationBasedAlgorithm idref="clusteringPSO"/>
    </algorithm>

## Cooperative Multi-swarm Data Clustering PSO
Create a normal data clustering PSO as described above. Set the window size and frequency. Add n + 1 of these standard
clustering PSOs to the muli-population algorithm, where n is the number of clusters required. The +1 is there because
we need an explorer swarm as the weakest swarm is reinitialised by the algorithm if all have converged.

XML:

    <algorithm id="cooperativePSO" class="clustering.CooperativePSO">
        <iterationStrategy class="pso.multiswarm.CooperativeMultiswarmIterationStrategy">
            <delegate class="pso.multiswarm.StandardClusteringMultiSwarmIterationStrategy"/>
        </iterationStrategy>
        <addStoppingCondition class="stoppingcondition.MeasuredStoppingCondition" target="6000"/>
        <addPopulationBasedAlgorithm idref="clusteringPSO"/>
        <addPopulationBasedAlgorithm idref="clusteringPSO"/>
        <addPopulationBasedAlgorithm idref="clusteringPSO"/>
        <addPopulationBasedAlgorithm idref="clusteringPSO"/>
    </algorithm>

## Specifying the problem

XML:

    <problems>
        <problem id="quantization" class="problem.QuantizationErrorMinimizationProblem" domain="R(-10:10)"/>
    </problems>

## Measurements:

ClusterCentroids is the mesurement that allows one to print the cluster centroids to a file. Using 
dimension="1" means that for all paricles, the centroid at index 1 will be printed to the file.
Note that particleA and particleB may have found the same centroids but they may have them in a different
order, so it is recommended for all centroids to be written to a file and a separate program to be used to clean
and average the centroids. The other measurements are more straight forward, these are the validity indexes 
that one may use.

XML:

    <measurements id="fitness" class="simulator.MeasurementSuite" resolution="10">
        <addMeasurement class="measurement.multiple.ClusterCentroids" dimension="1"/>
        <addMeasurement class="measurement.clustervalidity.DunnValidityIndex"/>
        <addMeasurement class="measurement.clustervalidity.DaviesBouldinValidityIndex"/>
        <addMeasurement class="measurement.clustervalidity.HalkidiVazirgiannisValidityIndex"/>
        <addMeasurement class="measurement.clustervalidity.RayTuriFavouringValidityIndex"/>
        <addMeasurement class="measurement.clustervalidity.RayTuriValidityIndex"/>
        <addMeasurement class="measurement.single.Fitness"/>
    </measurements>


If there are any questions join us on [irc chat](http://webchat.freenode.net/?channels=cilib).
