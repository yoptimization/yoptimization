---
title: "Connections"
last_updated: May 4, 2019
keywords: YopSystem, connections, YopConnection
sidebar: mydoc_sidebar
permalink: connections
toc: false
folder: gettingStarted/YopSystem
---
It is possible to connect systems using the `YopConnection` command. The purpose of this is to simplify creating an initial guess for non-convex optimal control problems. By connecting your system to a controller, it is possible to create an initial guess by simulation, even for an unstable system. Despite being the main purpose, connecting the system to a controller is not the only use for connections.

## Declaration
Connections are declared using the following syntax:
```matlab
c = YopConnection(arg1, arg2);
```
The input arguments `arg1`, and `arg2` can be any symbolic expression that can be formulated using the system variables `t`, `x`, `z`, `u`, `w`, `p`, `ode`, `ae`, and `y` (see [YopSystem](yopSystem#declaration) for a description of them). `arg1` and `arg2` can be vector valued as long as they have the same dimension. The output argument `c` is a handle to the connection.

## What a `YopConnection` is
A `YopConnection` is an algebraic constraint stating

$$ 0 = \texttt{arg1} - \texttt{arg2} $$

This means that the following declarations are equivalent:
```matlab
c1 = YopConnection(arg1, arg2);
c2 = YopConnection(arg1-arg2, zeros(size(arg1-arg2)));
c3 = YopConnection(zeros(size(arg1-arg2)), arg1-arg2);
```
