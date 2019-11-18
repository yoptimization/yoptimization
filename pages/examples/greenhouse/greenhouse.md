---
title: "Greenhouse Climate Control"
last_updated: June 2, 2019
keywords: example, Greenhouse, Growing, light, optimal, control, cultivation, willigenburg, external, input, interpolation
sidebar: mydoc_sidebar
permalink: greenhouse
folder: examples/greenhouse
toc: false
---

This problem is taken from the book: Optimal Control of Greenhouse Cultivation G. van Straten, R.J.C. van Ooteghem, L.G. van Willigenburg, E. van Henten.

The problem models the growth of crop in a greenhouse depending on the temperature and light. The problem is modeled with external input for the outside temperature and light coming in, and the control is a heating element inside the greenhouse.
The goal is to maximize the profit of the greenhouse which comes from the weight of crops minus the cost of heating.


## Problem formulation
The problem formulation is the following:

$$ J = - p_{5} x_1 + \int_{t_0}^{t_f} p_{4} u$$

$$ \dot{x}_1 = p_1 I x_2 $$

$$ \dot{x}_2 = p_2 (T_0 - x_2)+p_3 u $$

$$ I = max(0, 800 \sin( 4 \pi \frac{t}{t_f} - 0.65 \pi))$$

$$ T_0 = 15+10 \sin(4\pi \frac{t}{t_f} - 0.65\pi) $$

$$ p_1 = 7.5 \cdot 10^{-8}, \quad p_2 = 1, \quad p_3 = 0.1 $$

$$ p_4 = 4.55 \cdot 10^{-4}, \quad p_{5} = 136.4$$

$$ t_0 = 0 $$

$$ t_f = 48 $$

$$ x_1(0) = 0 $$

$$ x_2(0) = 10 $$

$$ 0 \leq u \leq 10 $$


Variable descriptions:

| variable  | Description                                    | Unit                  |
|-----------|------------------------------------------------|-----------------------|
| $$x_1$$   | Dry weight of the crop                         | $$\textrm{kgm}^{-2}$$ |
| $$x_2$$   | Temperature in the greenhouse                  | °C                    |
| $$I$$     | Intensity of the light entering the greenhouse | $$\textrm{Wm}^{-2}$$  |
| $$T_0$$   | Temperature outside the greenhouse             | °C                    |
| $$u$$     | Heat input from the heating system             | $$\textrm{Wm}^{-2}$$  |



## Yop implementation
This example shows how to model external input with the help of `YopInterpolant`.

{% include warning.html content="`YopInterpolant` assumes that the input is row vectors and might cause Matlab to crash if they're not." %}

```matlab
%% Greenhouse Climate Control, with external input
% Author: Dennis Edblom
sys = YopSystem('states', 2, 'controls', 1, ...
    'model', @greenhouseModel);
% Symbolic variables
t = sys.t;
x = sys.x;
u = sys.u;

%% Formulate optimal control problem
p4 = 4.55e-4;
p5 = 136.4;
tf = 48;

ocp = YopOcp();
ocp.min({ t_f(-p5*x(1)) '+' timeIntegral(p4*u) });
ocp.st(...
     'systems', sys, ...
     ... % Initial conditions
    { 0  '==' t_0(x(1)) }, ...
    { 10 '==' t_0(x(2)) }, ...
    ... % Constraints
    { 0  '<=' u '<=' 10 }, ...
    ... % Final conditions
    { tf '==' t_f(t)    } ...
    );
sol = ocp.solve('controlIntervals', 100);

%% Plot
% States, time and control
x1 = sol.signal(sys.x(1))';
x2 = sol.signal(sys.x(2))';
u = sol.signal(sys.u)';
t = sol.signal(sys.t)';

% External input for plots, comes from greenhouseModel
te = sys.y.te;
I  = sys.y.I;
T0 = sys.y.T0;

% Plot external inputs and control
figure(1);
plot(te,I./40,te,T0,t,u); axis([0 tf -1 30]);
xlabel('Time [h]');
ylabel('Heat input, temperatures & light');
legend('Light [W]','Outside temp. [oC]','Heat input [W]');
title('Optimal heating, outside temperature and light');

% Plot the optimal states
figure(2)
sf1=1200; sf3=60;
x3 = cumtrapz(t,p4*u); % Integral(pHc*u)
plot(t,[sf1*x1 x2 sf3*x3]); axis([0 tf -5 30]);
xlabel('Time [h]'); ylabel('states');
legend('1200*Dry weight [kg]','Greenhouse temp. [oC]','60*Integral(pHc*u dt) [J]');
title('Optimal system behavior and the running costs');

%% Model
function [dx, y] = greenhouseModel(t, x, u)
% Constants
p1 = 7.5e-8;
p2 = 1;
p3 = 0.1;
tf = 48;
% External inputs: [time, sunlight, outside temperature]
te  = (-1:0.2:49);
I = max(0, 800*sin(4*pi*te/tf-0.65*pi));
T0 = 15+10*sin(4*pi*te/tf-0.65*pi);

% Extract external inputs from table tue through interpolation
d1 = YopInterpolant(te, I);
d2 = YopInterpolant(te,T0);

dx1 = p1*d1(t)*x(2);
dx2 = p2*(d2(t)-x(2))+p3*u;
dx = [dx1; dx2];
y.te = te;
y.I = I;
y.T0 = T0;
end
```




## Plots
{% include image.html file="example_images/greenhouse/greenhouseStates.svg" alt="greenhouseStates" caption="Greenhouse states" %}

{% include image.html file="example_images/greenhouse/greenhouseControl.svg" alt="greenhouseControl" caption="Greenhouse control and external input" %}
