---
title: "Getting started with Yop"
last_updated: May 4, 2019
keywords: Getting stared, basics, YopSystem, YopSimulator, YopOcp
sidebar: docs_sidebar
permalink: gettingStarted
folder: gettingStarted
toc: false
---
To get started with Yop you can either start from examples, you can visit the Yop [examples page](examples). You can also read more about Yop under the different categories.

# Basics
The basics of Yop can be understood from the three base classes `YopSystem`, `YopOcp`, and `YopSimulator`.

The `YopSystem` class is used for representing systems, or models. This is necessary in order to formulate and solve optimal control problems (OCPs), and for setting up simulations. More information about `YopSystem` is found at the [`YopSystem`](yopSystemOverview) page.

The `YopOcp` class is used for formulating and solving optimal control problems. More information on this class is found at the [`YopOcp`](yopOcp) page.

The `YopSimulator` class is used for simulations. The main purpose of this class is to be used as a way of finding an initial guess for non-convex optimal control problems. The class supports semi-explicit DAEs of differential index 1. The main idea of supporting DAEs of index one is the ability to connect ODEs consisting of a system and a controller via an algebraic constraint. More info is found at the [`YopSimulator`](yopSimulator) page.

## Links `YopSystem`
[Overview](yopSystemOverview)

[Declaring a YopSystem](yopSystem)

[Connecting systems](connections)

[YopSystem Limitations](yopSystemLimitations)

## Links `YopOcp`
[Overview](yopOcp)

[Formulating optimal control problems](formulatingOptimalControlProblems)

[How to provide an initial guess](initialGuess)

[How to solve optimal control problems](solvingOcps)

## Links `YopSimulator`
[Overview](simulationOverview)

[Supported systems](yopSimulator#supported-systems)

[Setting up a simulator](yopSimulator#declaring-a-yopsimulator)

[Running a simulation](yopSimulator#simulating)

[Symbolic plotting](yopSimulator#numerical-results-by-symbolic-plotting)
