---
layout: docs
title: "Cascade Neural Network Architecture"
description: ""
group: "user-docs"
tags: [neural network, cascade network, pso]
---

The functionality of the feed-forward neural network in CILib has been adapted to give the functionality
of a cascade network. A cascade network consists of one input, one hidden and one output layer.
The neurons in the hidden layer are connected in such a manner that each hidden neuron receives inputs
from the entire input layer, as well as from all the hidden neurons that were added before it.
Neurons in the output layer receives inputs from both the entire input layer and the entire output layer.
The intent of this type of architecture, is to make it possible to incrementally grow the architecture,
by adding a neuron to the hidden layer, until the architecture's size is optimal.

The cascade network can be trained by a PSO. However, this is still under investigation.

# Implementation Details

## Cascade Architecture

- CascadeLayerBuilder: a LayerBuilder that builds a cascading hidden layer. It ensures that each
Neuron has an amount of weights equal to the number of neurons before it in the hidden layer, plus
the size of the input layer.

- CascadeArchitectureBuilder: an ArchitectureBuilder that builds cascade networks. It makes use
of a standard feedforward configuration (with one input layer, one hidden layer and one output layer),
except that it uses CascadeLayerBuilder to construct a cascading hidden layer. It also ensures that
only the input layer contains a bias unit (since both the hidden and output layer will have access
to all the neurons in the input layer).

- CascadeVisitor: an ArchitectureOperationVisitor that propagates inputs in a cascading fashion.
It constructs a consolidated Layer by adding all the Neurons of the input layer (in order) and then
adding all of the Neurons of the hidden layer (in order). This consolidated layer is then used as
input for both the hidden layer and the output layer. Since each Neuron, receiving this input,
only has a number of weights corresponding to the number of inputs it should receive, each Neuron
will only use the inputs relevant to it.

## Training a Cascade Network with a Dynamic PSO

- CascadeNetworkexpansionReactionStrategy: an EnvironmentChangeResponseStrategy that expands the
size of the CascadeNetwork by one hidden neuron. It does this by changing the size parameter of the
LayerConfiguration corresponding to the hidden layer and then using CascadeArchitechureBuilder to
build a new cascade network. Afterwards, it expands the dimensions of the search space of the
particles. It adds one new dimension, for each weight, to the particles' vectors in the positions
corresponding to the new weights. These new dimensions are given an initial value of "Not a Number"
to indicate that they need to be initialised by another EnvironmentChangeResponseStrategy.

- InitialiseNaNElements: an EnvironmentChangeResponseStrategy that initialises any dimensions within
particles that hold the value "Not a Number". Elements in the particle's position vector are randomly
initialised within that dimension's boundaries. Elements in the particle's best position vector are
set to the same value of the corresponding element in the position vector. Elements in the velocity
vector are set to zero.

- ReinitialiseCascadeNetworkOutputWeightsReactionStrategy: an EnvironmentChangeResponseStrategy
that reinitialises any dimensions within the particles that correspond to the output weights
of a cascade network. Elements in the particle's position vector are randomly initialised within
that dimension's boundaries. Elements in the particle's best position vector are left as is.
Elements in the velocity vector are set to zero.

- ReevaluationReactionStrategy: an EnvironmentChangeResponseStrategy that reevaluates the fitness of
all the particles.

- MultiReactionStrategy: a EnvironmentChangeResponseStrategy that executes multiple
EnvironmentChangeResponseStrategies. It simply iterates through an ordered list of responses.
It is used, in this context, to combine CascadeNetworkexpansionReactionStrategy with one or more
responses that will initialise new weights (and potentially old weights). After which,
ReevaluationReactionStrategy is used to calculate new fitnesses.

# XML Examples

## Specifying a cascade network as the problem

An ARFF data set is loaded.
A CascadeVisitor is used as the operation visitor for the neural network.
The Architecture's builder is set to a CascadeArchitectureBuilder.
The architecture needs to be given exactly three LayerConfigurations.
The architecture is given a size of 4 for the input layer (by default, a fifth bias unit will be added).
The architecture's hidden layer is given an initial size of 0 and the activation function is set to be Sigmoids.
The architecture's output layer is given a size of 1.
The output layer is manually set to be constructed by the PrototypeFullyConnectedLayerBuilder,
while the hidden layer is automatically set to be constructed by the CascadeLayerBuilder.

XML:

	<problems>
		<problem id="nnProblem" class="problem.NNDataTrainingProblem" trainingSetPercentage="0.7" generalizationSetPercentage="0.3">
			<dataTableBuilder class="io.DataTableBuilder">
				<dataReader class="io.ARFFFileReader" sourceURL="library/src/test/resources/datasets/iris.arff"/>
			</dataTableBuilder>
			<neuralNetwork class="nn.NeuralNetwork">
				<operationVisitor class="nn.architecture.visitors.CascadeVisitor"/>
				<architecture class="nn.architecture.Architecture">
					<architectureBuilder class="nn.architecture.builder.CascadeArchitectureBuilder">
						<addLayer class="nn.architecture.builder.LayerConfiguration" size="4"/>
						<addLayer class="nn.architecture.builder.LayerConfiguration" size="0">
							<activationFunction class="functions.activation.Sigmoid" />
						</addLayer>
						<addLayer class="nn.architecture.builder.LayerConfiguration" size="1"/>
						<layerBuilder class="nn.architecture.builder.PrototypeFullyConnectedLayerBuilder" domain="R(-3:3)" />
					</architectureBuilder>
				</architecture>
			</neuralNetwork>
		</problem>
	</problems>

## Training the cascade network with a PSO

A dynamic PSO is initialised with ten particles.
SynchronousIterationStrategy iteration strategy is used.
Detection strategy is set to trigger every ten iterations.
Multiple responses are used:
the first expands the cascade network,
the second initialises the new uninitialised weights,
the third reinitialises all the weights of the output units,
and the fourth reevaluates all the particles.
Algorithm stops execution after one hundred iterations.

XML:

	<algorithms>
		<algorithm id="pso" class="pso.PSO">
			<initialisationStrategy class="algorithm.initialisation.ClonedPopulationInitialisationStrategy">
				<entityNumber value="10"/>
				<entityType class="pso.dynamic.DynamicParticle"/>
			</initialisationStrategy>
			<iterationStrategy class="pso.dynamic.DynamicIterationStrategy">
				<iterationStrategy class="pso.iterationstrategies.SynchronousIterationStrategy"/>
				<detectionStrategy class="pso.dynamic.detectionstrategies.PeriodicDetectionStrategy" iterationsModulus="10"/>
				<responseStrategy class="pso.dynamic.responsestrategies.MultiReactionStrategy">
					<addResponseStrategy class="pso.dynamic.responsestrategies.CascadeNetworkExpansionReactionStrategy"/>
					<addResponseStrategy class="pso.dynamic.responsestrategies.InitialiseNaNElementsReactionStrategy"/>
					<addResponseStrategy class="pso.dynamic.responsestrategies.ReinitialiseCascadeNetworkOutputWeightsReactionStrategy"/>
					<addResponseStrategy class="pso.dynamic.responsestrategies.ReevaluationReactionStrategy" reevaluationRatio="1.0"/>
				</responseStrategy>
			</iterationStrategy>
			<addStoppingCondition class="stoppingcondition.MeasuredStoppingCondition" target="100"/>
		</algorithm>
	</algorithms>

If there are any questions join us on [irc chat](http://webchat.freenode.net/?channels=cilib).
