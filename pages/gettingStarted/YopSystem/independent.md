---
title: "The independent variable"
last_updated: May 4, 2019
keywords: YopSystem, system, independent, variable
sidebar: mydoc_sidebar
permalink: independent
toc: false
folder: gettingStarted/YopSystem
---
The independent variable in Yop is treated in a special way using the class `YopIndependentVariable`. This way all `YopSystem`s share the same independent variable. This is of practical value to the user when plotting or obtaining numerical results from a simulation or optimal control problem, since it means that the independent variable of any `YopSystem` can be used. It is also possible to run `YopIndependentVariable.getIndependentVariable`.
