---
title: "Yop example overview"
last_updated: June 14, 2018
keywords: example, overview
sidebar: mydoc_sidebar
permalink: examples
toc: false
folder: examples
---



### Bryson-Denham problem
The Bryson-Denham problem is a classical minimum energy optimal control problem.
* [Bryson-Denham](brysonDenham): This example is a good starting point for the Yop toolbox.

### Goddard rocket
The Goddard rocket problem is an optimal control problem where the goal is to maximize the altitude of a vertically launched rocket.

* [Basic rocket](goddardRocket): Basic example of the Goddard rocket.
* [Goddard rocket final time free](goddardRocketFreeTf): The Goddard rocket with a free final time.
* [Goddard rocket final time fixed](goddardRocketFixedTf): The same model as in the example with free final time but this time with a fixed final time.
* [Goddard rocket landing](goddardLanding): A multi-phase problem to land the Goddard rocket.


### A Linear Problem With Bang Bang Control
Simple linear problem.
* [Linear problem with bang-bang control](bangFreeTf): Shows an alternative way of setting a model to the `YopSystem`.


### Greenhouse Climate Control
Controlling the temperature in a greenhouse to maximize yield. This problem introduces external input.

* [Greenhouse](greenhouse): Optimal control problem with external input using `YopInterpolant`.

### Isoperimetric Constraint Problem
Example showing how to implement isoperimetric constraints.

* [Isoperimetric constraint problem](isoConstraint): Simple example with isoperimetric constraint.

### Nondifferentiable system
Example of a nondifferentiable system.

* [Nondifferentiable system](nonDiff): Showing how to model a nondifferentiable system with the casadi function `if_else`.
