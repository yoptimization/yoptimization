---
title: "A Linear Problem With Bang Bang Control"
last_updated: February 26, 2019
keywords: example, Bang Bang, control, linear
sidebar: mydoc_sidebar
permalink: bangFreeTf
folder: examples/bangFreeTf
toc: false
---

This problem is taken from the [PROPT manual, example 10.](https://tomopt.com/docs/propt/tomlab_propt011.php){:target="_blank"}

This problem shows an alternative way of implementing a model in Yop.

## Problem formulation
The formulation is the following:

$$\min t_f$$

$$\dot{x}_1 = x_2$$

$$\dot{x}_2 = u$$

$$x_1(0) = 0$$

$$x_2(0) = 0$$

$$x_1(t_f) = 300$$

$$x_2(t_f) = 0$$

$$-2 < u < 1$$


## Yop implementation
Below follows the code for the solution of the problem with Yop.

{% include note.html content="The model is an vector with the symbolic variables and can be set directly to the system with `sys.set('ode',dx)`." %}

### Mail file
```matlab
%% A Linear Problem With Bang Bang Control
% Create the Yop system
sys = YopSystem('states', 2, 'controls', 1);
% Symbolic variables
t = sys.t;
x = sys.x;
u = sys.u;

% Model
xdot = [x(2); u];
sys.set('ode',xdot)

% Formulate optimal control problem
ocp = YopOcp();
ocp.min({ t_f( t ) });
xf = [300; 0];
ocp.st(...
     'systems', sys, ...
     ... % Initial conditions
    {  0  '==' t_0(  t   )   }, ...
    {  0  '==' t_0( x(1) )   }, ...
    {  0  '==' t_0( x(2) )   }, ...
    ... % Constraints
    {-inf '<=' x(1) '<=' inf }, ...
    {-inf '<=' x(2) '<=' inf }, ...
    { -2  '<='  u   '<=' 1   }, ...
    ... % Final conditions
    { 300 '==' t_f( x(1) )   }, ...
    {  0  '==' t_f( x(2) )   } ...
    );

% Solving the OCP
sol = ocp.solve('controlIntervals', 30);

%% Plot the results
figure(1)
subplot(211); hold on
sol.plot(sys.t, sys.x(1))
xlabel('Time')
ylabel('Position')

subplot(212); hold on
sol.plot(sys.t, sys.x(2))
xlabel('Time')
ylabel('Velocity')

figure(2); hold on
sol.stairs(sys.t, sys.u)
xlabel('Time')
ylabel('Acceleration (Control)')
```

## plots
{% include image.html file="example_images/bangFreeTf/bangFreeTfStates.svg" alt="bangFreeTfStates" caption="Bang-Bang Free Time States" %}

{% include image.html file="example_images/bangFreeTf/bangFreeTfControl.svg" alt="bangFreeTfControl" caption="Bang-Bang Free Time Control" %}
