---
title: "Goddard Rocket Maximum Ascent, Final Time Free"
last_updated: June 2, 2019
keywords: example, Goddard, rocket, spaceX, maximum, ascent, final, time, free, tf, NASA, space
sidebar: mydoc_sidebar
permalink: goddardRocketFreeTf
folder: examples/goddardRocket
toc: false
---

This example is taken from the [PROPT manual example 44.](https://tomopt.com/docs/propt/tomlab_propt045.php){:target="_blank"}
The example is simular to the previous example [Goddard rocket maximum ascent](goddardRocket) but with slightly different model parameters.

{% include image.html file="example_images/goddard/goddardRocketFreeTf.gif" alt="goddardRocketFreeTfAnimation" caption="Goddard rocket animation, final time free" %}

## Problem formulation
The problem formulation is the following:

$$\max \: h(t_{f})$$

$$\dot{v} = \frac{F \cdot c-D(h,v)}{m}-g(h)$$

$$\dot{h} = v$$

$$\dot{m} = - F$$

$$D(h,v) = D_0 v^2 exp( -\beta h )$$

$$g(h) = g_0\Big(\frac{r0}{r0+h} \Big)^2$$

$$m(0) = 214.839$$

$$m(t_f) = 67.9833$$

$$v(0) = 0$$

$$h(0) = 0$$

$$g_0 = 9.81$$

$$r_0 = 6.371e6$$

$$0 \leq F \leq 9.525515$$

$$D_0 = 0.01227$$

$$c = 2060$$

$$\beta = 0.145e-3$$

The states are the rocket velocity $$v$$, the altitude from the ground $$h$$ and mass of the rocket $$m$$. The rocket burns fuel and expels it out the nozzle creating thrust, in the model the fuel flow rate $$F$$ is the control and the ratio between thrust and change in mass is $$T = F \cdot c$$.


## Yop implementation
```matlab
%% Goddard Rocket, Maximum Ascent
% Author: Dennis Edblom
% Create the Yop system
sys = YopSystem('states', 3, 'controls', 1, ...
    'model', @goddardRocketModel);
% Symbolic variables
time = sys.t;

% Rocket signals (symbolic)
rocket = sys.y.rocket;

m0 = 214.839;
mf = 67.9833;
Fm = 9.525515;
% Formulate optimal control problem
ocp = YopOcp();
ocp.max({ t_f( rocket.height ) });
ocp.st(...
     'systems', sys, ...
    ... % initial conditions
    { 0  '==' t_0(time)            }, ...
    { 0  '==' t_0(rocket.velocity) }, ...
    { 0  '==' t_0(rocket.height)   }, ...
    { m0 '==' t_0(rocket.mass)     }, ...
    ... % Constraints
    {  0 '<=' t_f( time)          '<=' inf }, ...
    {  0 '<=' rocket.velocity     '<=' inf }, ...
    {  0 '<=' rocket.height       '<=' inf }, ...
    { mf '<=' rocket.mass         '<=' m0  }, ...
    {  0 '<=' rocket.fuelMassFlow '<=' Fm  } ...
    );

% Solving the OCP
sol = ocp.solve('controlIntervals', 60);

%% Plot the results
figure(1)
subplot(311); hold on
sol.plot(time, rocket.velocity)
xlabel('Time')
ylabel('Velocity')

subplot(312); hold on
sol.plot(time, rocket.height)
xlabel('Time')
ylabel('Height')

subplot(313); hold on
sol.plot(time, rocket.mass)
xlabel('Time')
ylabel('Mass')

figure(2); hold on
sol.stairs(time, rocket.fuelMassFlow)
xlabel('Time')
ylabel('F (Control)')

%% Model
function [dx, y] = goddardRocketModel(t, x, u)
% States and control
v = x(1);
h = x(2);
m = x(3);
F = u;

% Parameters
D0   = 0.01227;
beta = 0.145e-3;
c    = 2060;
g0   = 9.81;
r0   = 6.371e6;

% Drag and gravity
D   = D0*exp(-beta*h);
F_D = sign(v)*D*v^2;
g   = g0*(r0/(r0+h))^2;

% Dynamics
dv = (F*c-F_D)/m-g;
dh = v;
dm = -F;
dx = [dv;dh;dm];

% Signals y
y.rocket.velocity     = v;
y.rocket.height       = h;
y.rocket.mass         = m;
y.rocket.fuelMassFlow = F;
y.drag.coefficient    = D;
y.drag.force          = F_D;
y.gravity             = g;
end
```

## Plots

{% include image.html file="example_images/goddard/goddard44States.svg" alt="goddardRocketFreeTfStates" caption="Goddard rocket states" %}

{% include image.html file="example_images/goddard/goddard44Control.svg" alt="goddardRocketFreeTfControl" caption="Goddard rocket control" %}
