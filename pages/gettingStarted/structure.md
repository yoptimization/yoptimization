---
title: "Yop structure"
last_updated: May 3, 2019
keywords: Getting stared, basics, structure
sidebar: mydoc_sidebar
permalink: structure
toc: true
folder: gettingStarted
---
The primary purpose of Yop is to be a tool for numerical optimal control. This typically involve a model of a dynamical system, an optimal control problem, and if the problem is non-convex, an initial guess. To meet this, Yop is built around three base classes:  [`YopSystem`](yopSystem), [`YopOcp`](yopOcp), and [`YopSimulator`](yopSimulator). The `YopSystem` class is a symbolic representation of a model. It is used as input to optimal control problems and simulations. `YopOcp` is the optimal control problem (OCP) class and is used for formulating and solving optimal control problems. The `YopSimulator` class is used for simulation with the main purpose of providing an initial guess and validating optimal control results. 
