---
title: "Isoperimetric Constraint Problem"
last_updated: June 3, 2019
keywords: example, isoperimetric, constraint
sidebar: mydoc_sidebar
permalink: isoConstraint
folder: examples/isoConstraint
toc: false
---

This problem is taken from the [PROPT manual, example 54.](https://tomopt.com/docs/propt/tomlab_propt055.php){:target="_blank"}
The example shows how to implement a isoperimetric constraint.

The formulation is the following:

$$\min \int_0^1 x^2 dt$$

$$\dot{x} = -\sin(x) + u$$

$$\int_0^1 u^2 dt = 10$$

$$x(0) = 1$$

$$x(1) = 0$$

### Yop implementation
The problem is solved by implementing another state that keeps track of $$\int u^2 dt$$ and force it to be equal to 10.
```matlab
%% Isoperimetric Constraint Problem
% Author: Dennis Edblom
sys = YopSystem('states', 2, 'controls', 1);
% Symbolic variables
t = sys.t;
x = sys.x;
u = sys.u;

% Model
xdot = [-sin(x(1)) + u; u^2];
sys.set('ode',xdot)

%% Formulate optimal control problem
ocp = YopOcp();
ocp.min({ LagrangeTerm( x(1) ) });
ocp.st(...
     'systems', sys, ...
     ... % Initial conditions
    { 0 '=='  t_0(  t   )    }, ...
    { 1 '=='  t_0( x(1) )    }, ...
    { 0 '=='  t_0( x(2) )    }, ...
    ... % Constraints
    { -4  '<='  u   '<='  4  }, ...
    { -10 '<=' x(1) '<=' 10  }, ...
    {-inf '<=' x(2) '<=' inf }, ...
    ... % Final conditions
    { 1  '<=' t_f(t)    '<=' 1 }, ...
    { 0  '<=' t_f(x(1)) '<=' 0 }, ...
    { 10 '==' t_f(x(2))        } ...
    );

sol = ocp.solve('controlIntervals', 30);

%% Plot the results
figure(1)
subplot(211); hold on
sol.plot(sys.t, sys.x(1))
xlabel('Time')
ylabel('x1')

subplot(212); hold on
sol.plot(sys.t, sys.x(2))
xlabel('Time')
ylabel('x2')

figure(2); hold on
sol.stairs(sys.t, sys.u)
xlabel('Time')
ylabel('u')
```

## Plots

{% include image.html file="example_images/isoConstraint/isoConstraintStates.svg" alt="isoConstraintStates" caption="Isoperimetric Constraint states" %}

{% include image.html file="example_images/isoConstraint/isoConstraintControl.svg" alt="isoConstraintControl" caption="Isoperimetric Constraint control" %}
