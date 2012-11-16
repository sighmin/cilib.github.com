---
layout: post
title: "Niching Makeover: part 1"
description: ""
category:
tags: []
---

The niching package has recently had a major refactor. This post and a few later
ones will explain the changes and how to use them.

### Going Functional

A functional approach has been taken with regards to the refactor. This means that
instead of objects being changed in place (e.g. `particle.setCandidateSolution(newSolution)`)
functions are used to create new objects with the changes in place. Currently CIlib
is not completely set up for this so to emulate it first clone the object then make
the changes to the cloned object.

The reason for taking a functional approach is to reduce mutable state. Mutable
state makes it harder to debug code because objects can be changed in multiple
places. Functional programs work with immutable data meaning that a copy of the
object is made when they need to be changed.

Functional programming is a different paradigm to the way the rest of CIlib is
implemented and so a few definitions are presented here to make the transition
easier. For those who want to make changes to the code a java library,
Functional Java (FJ), was used to implement the changes.

### Definitions

* #### Function:
    A function is a unit of code that transforms an input into an output. The FJ
    library has a number of function classes for different numbers of inputs e.g.

        F<A, Z>

    is a function that takes an A as input and outputs a Z and

        F2<A, B, Z>
        F3<A, B, C, Z>
        ...
        F8<A, B, C, D, E, F, G, H, Z>

    take in an `A ... H` and return a `Z`. To apply the functions a method, `f(...)`,
    is called. Here is an example that takes a particle and returns its candidate
    solution.

        F<Particle, Vector> solution = new F<Particle, Vector>() {
            public Vector f(Particle p) {
                return (Vector) p.getCandidateSolution();
            }
        }

    and to call it for a particular particle

        solution.f(particularParticle)

* #### Tuple/Product:
    A tuple is a collection of heterogeneous data. It allows functions to return
    more than one result. Using a similar naming convention to functions, FJ's
    tuples are called

        P1<A>
        P2<A, B>
        ...
        P8<A, B, C, D, E, F, G, H>

    To create a tuple use the static method `P.p(...)` e.g. to create a tuple of a
    particle with an associated score use

        P2<Particle, Double> x = P.p(particle, score)

    To access the individual elements of a tuple use the `_n()` methods, where n
    is the position of the element in the tuple e.g.

        Particle particle = x._1()
        Double score = x._2()

* #### Lists:
    Lists in FJ have far more functionality (no pun intended) than Java's collections.
    Here are a few operations that can be performed on them that apply the functional
    paradigm.

    -- **map**: this applies a function to each element and returns a new list consisting
    of the output of the given function. e.g.

        List<Double> numbers = List.list(-1.0, 0.0, 1.0);

        F<Double, Boolean> ltZero = new F<Double, Boolean>() {
            public Boolean f(Double p) {
                return p < 0;
            }
        }

        List<Boolean> output = numbers.map(ltZero);

    The contents of `output` is `[false, false, true]`

    -- **filter/removeAll**: given a `List<A>`, `l`,  and a function `F<A, Boolean>`,
    `f`, `l.filter(f)` returns a list of all the elements in `l` which, when `f`
    is applied to the element, returns `true`. Similarly, `l.removeAll(f)`
    returns a list for which the result of `f` is `false`. Note that the input of
    the function, `A`, is the same type as the contents of the list.

    -- **foldLeft**: given `List<A> l; F2<B, A, B> f; A a`, `l.foldLeft(f, a)`
    will apply `f` to `a` and the first element of `l` which results in a `B`.
    `f` is the applied to that result and the next element of `l` and so on until
    the list is exhausted. This operation is also known as reduce in other
    langueages/frameworks because it reduces a list of elements to one value.

    -- **head/tail**: `head` returns the first element of the list and `tail` returns
    a new list containing every element except the `head` of the list. Similarly `last`
    returns the last element and `init` returns all but the last element of a list.

### Functions

In functional programming the user specifies what they want e.g. `list.filter(ltZero)`)
as opposed to how to get it e.g.

    for (Double n : numbers) {
        if (n < 0) {
            newList.add(n);
        }
    }

Another thing is that functions can be composed, so the output of one function can
serve as the input for another function like the following

    F<Algorithm, OptimisationSolution> solution = new F<Algorithm, OptimisationSolution>() {
        public OptimisationSolution f(Algorithm a) {
            return a.getBestSolution();
        }
    }

    F<OptimisationSolution, Boolean> fitnessThreshold(final double threshold) {
        return new F<OptimisationSolution, Boolean>() {
            public Boolean f(OptimisationSolution s) {
                return s.getFitness().getValue() < threshold;
            }
        };
    }

    List<Algorithm> algorithms = ...;
    algorithms.filter(solution.andThen(fitnessThreshold(0.5)));

Here `solution` is composed with `fitnessThreshold`. An algorithm from the list
is passed to `solution` which returns an `OptimisationSolution` which gets passed
to `fitnessThreshold(0.5)` which determines if the solution is less than 0.5.
The result is a list of algorithms whose best solution is less than 0.5. If one
forgets most of their grammar rules it almost reads like English :)

One thing most people will notice is that the specification can get very verbose.
This is a consequence of using Java and its generics and not of functional
programming itself e.g. Scala can condense the above to one line per function.

#### What's next?

In the next post the niching functions and structures will be introduced.
