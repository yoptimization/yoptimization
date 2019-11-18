---
title: "Optimal control"
last_updated: May 5, 2019
keywords: YopSystem, YopSimulator, YopOcp, optimal control problems, initial guess
sidebar: mydoc_sidebar
permalink: yopOcp
toc: false
folder: gettingStarted/YopOcp
---
Optimal control problems are formulated using the `YopOcp` class. How to declare, formulate and solve optimal control problems are found under the different topics:

### Formulating optimal control problems
An in-depth description on how to formulate optimal control problems are found at the [OCP formulation page](formulatingOptimalControlProblems).

#### Links

[How to use t_0 and t_f](formulatingOptimalControlProblems#two-key-functions-t_f-and-t_0)

[Declaring optimal control problems](formulatingOptimalControlProblems#declaring-optimal-control-problems)

[Declaring the objective function](formulatingOptimalControlProblems#declaring-the-objective-function)

[Declaring the constraints](formulatingOptimalControlProblems#declaring-constraints-using-st)

[How to formulate individual constraints](formulatingOptimalControlProblems#constraints)

[Information on box constraints](formulatingOptimalControlProblems#box-constraints)

[Strict path constraints](formulatingOptimalControlProblems#strict-path-constraints)

[Problem scaling](formulatingOptimalControlProblems#scaling)

[Multi-phase problems](formulatingOptimalControlProblems#multi-phase-problems)


### Initial guess
To be able to solve non-convex optimal control problems an initial guess may be necessary. For a description on how to provide and initial guess, see the [initial guess page](initialGuess).

#### Links
[Omitting to provide an initial guess](initialGuess)

[Using simulation to find an initial guess](initialGuess#using-simulation)

[Using an old solution as initial guess](initialGuess#using-a-solution-to-another-optimal-control-problem)

[Providing a general initial guess](initialGuess#using-yopinitialguess)

[Setting the initial guess to a fixed value](initialGuess#fixed-guess-value)

### Solving optimal control problems
How to solve optimal control problems, how the results are obtained, and the available settings are found at the [solving optimal controls problems page](solvingOcps).

#### Links
[Running the solve command](solvingOcps#running-solve)

[How to obtain the results](solvingOcps#how-the-results-are-obtained)

[Symbolic plotting](solvingOcps#plotting)

[Symbolic signals](solvingOcps#signals)

[Speed-up by re-parameterizing the problem](solvingOcps#build-parameterize-optimize)

[An example of re-parametrization](solvingOcps#example-of-re-parametrization-for-speed-up)
