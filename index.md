---
title: "Yop - A MATLAB Toolbox for Numerical Optimal Control based on CasADi"
keywords: sample homepage
sidebar: mydoc_sidebar
permalink: index.html
toc: false
---
{% include note-minimal.html content="**For IFAC reviewers:** We are currently working on documentation and final testing of the new version presented in the extended abstract for IFAC WC 2020. Until this is finalized the website will have the old syntax. We expect to be finished with this by February 2020. We apologize for the inconvenience. In the meantime the code on the website is runnable and can be downloaded, and should still provide a glimpse of what the new version will be able to achieve. " %}


Yop is a MATLAB Toolbox for numerical optimal control. It utilizes CasADi to interface to integrators and nonlinear optimization solvers, and thereof its name, Yop - Yet Another Optimal Control Problem Parser.

### Getting started
To get started with Yop you first need to install [Yop](install) and [CasADi](https://web.casadi.org/get/){:target="_blank"}.

<a target="_self" class="noCrossRef" href="{{ "install"}}"><button type="button" class="btn btn-default" aria-label="Left Align"><span class="glyphicon glyphicon-download-alt" aria-hidden="false"></span> Download Yop </button></a>

To get a more in-depth view of Yop you can visit the [getting started page](gettingStarted). Or you can try out any of the [examples](examples).

### A Yop example - Goddard rocket problem
The Goddard rocket problem is an optimal control problem where the goal is to maximize the altitude of a vertically launched rocket, using thrust as control.
The example is found on the [Goddard rocket page](goddardRocket). An illustration of the solution is found below. The problem formulation and Yop implementation are also found below.

{% include image.html file="example_images/goddard/goddard43.gif" alt="goddard43Animation" caption="Goddard Rocket Maximum Ascent Animation" %}

#### Problem formulation
The problem formulation is the following:

$$\max \: h(t_{f}) $$

$$\text{s.t} \,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,$$

$$\dot{v}(t) = \frac{T(t)-D(h(t),v(t))}{m(t)}-g(h(t))$$

$$\dot{h}(t) = v(t)$$

$$\dot{m}(t) = - \frac{T(t)}{c}$$

$$D(h(t),v(t)) = D_0 e^{ -\beta \Big( \frac{h(t)-h(0)}{h(0)} \Big)} v^2(t) $$

$$g(h) = g_0\Big(\frac{h(0)}{h(t)} \Big)^2$$

$$g_0 = 1, D_0 = 0.5*620, c = 0.5, \beta = 500$$

$$v(0) = 0$$

$$h(0) = 1$$

$$m(0) = 1$$

$$0.6 \leq m(t) \leq 1$$

$$0 \leq T(t) \leq 3.5$$

The states are the rocket velocity $$v(t)$$, the altitude from the center of the earth $$h(t)$$ and mass of the rocket $$m(t)$$. The rocket burns fuel and expels it out the nozzle creating thrust, in the model thrust $$T(t)$$ is the control and the ratio between thrust and change in mass is $$\dot{m}(t) = -\frac{T(t)}{c}$$.

#### Yop implementation
```matlab
%% Goddard Rocket, Maximum Ascent
% Author: Dennis Edblom
sys = YopSystem(...
    'states', 3, ...
    'controls', 1, ...
    'model', @goddardModel ...
    );
time = sys.t;

% Rocket signals (symbolic)
rocket = sys.y.rocket;

% Formulate optimal control problem
ocp = YopOcp();
ocp.max({ t_f( rocket.height ) });
ocp.st(...
     'systems', sys, ...
     ... % Initial conditions
    {   0  '==' t_0( time )              }, ...
    {   1  '==' t_0( rocket.height   )   }, ...
    {   0  '==' t_0( rocket.speed    )   }, ...
    {   1  '==' t_0( rocket.fuelMass )   }, ...
    ... % Constraints
    {   0  '<=' t_f( time )     '<=' inf  }, ...
    {   1  '<=' rocket.height   '<=' inf  }, ...
    { -inf '<=' rocket.speed    '<=' inf  }, ...
    {  0.6 '<=' rocket.fuelMass '<='  1   }, ...
    {   0  '<=' rocket.thrust   '<=' 3.5  } ...
    );

% Solving the OCP
sol = ocp.solve('controlIntervals', 100);

% Plot the results
figure(1)
subplot(311); hold on
sol.plot(time, rocket.speed)
xlabel('Time')
ylabel('Speed')

subplot(312); hold on
sol.plot(time, rocket.height)
xlabel('Time')
ylabel('Height')

subplot(313); hold on
sol.plot(time, rocket.fuelMass)
xlabel('Time')
ylabel('Mass')

figure(2); hold on
sol.stairs(time, rocket.thrust)
xlabel('Time')
ylabel('Thrust (Control)')

%% Model implementation
function [dx, y] = goddardModel(t, x, u)
% States and controls
v = x(1);
h = x(2);
m = x(3);
T = u;

% Parameters
c = 0.5;
g0 = 1;
h0 = 1;
D0 = 0.5*620;
b = 500;

% Drag
g = g0*(h0/h)^2;
D = D0*exp(-b*h);
F_D = D*v^2;

% Dynamics
dv = (T-sign(v)*F_D)/m-g;
dh = v;
dm = -T/c;
dx = [dv;dh;dm];

% Signals y
y.rocket.speed = v;
y.rocket.height = h;
y.rocket.fuelMass = m;
y.rocket.thrust = T;
y.drag.coefficient = D;
y.drag.force = F_D;
y.gravity = g;
end
```


#### Plots

{% include image.html file="example_images/goddard/goddard43States.svg" alt="goddard43States" caption="Goddard Rocket Maximum Ascent States" %}

{% include image.html file="example_images/goddard/goddard43Control.svg" alt="goddard43Control" caption="Goddard Rocket Maximum Ascent Control" %}


### Another Yop example - Bryson-Denham problem
The Bryson-Denham problem is a classical minimum energy optimal control
problem. This problem can be found on the [Bryson-Denham page](brysonDenham). An illustration of the problem can be found below. The problem formulation and Yop implementation can also be found below.

{% include image.html file="example_images/bryson/testAnimatedBryson2.gif" alt="BrysonAnimation" caption="Bryson-Denham Problem Visualization" %}



#### Problem formulation

The formulation is the following:

$$\min_{a(t)} \frac{1}{2} \int_0^1 a(t)^2 dt$$

$$\dot{v}(t) = a(t) = u(t)$$

$$\dot{x}(t) = v(t)$$

$$v(0)=-v(1)=1$$

$$x(0)=x(1)=0$$

$$x(t) \leq l = \frac{1}{9}$$

where $$u$$ is the control signal.


#### Yop implementation

```matlab
%% Bryson Denham
% Author: Dennis Edblom
% Create the Yop system
bdSystem = YopSystem(...
    'states', 2, ...
    'controls', 1, ...
    'model', @trolleyModel ...
    );

time = bdSystem.t;
trolley = bdSystem.y;

ocp = YopOcp();
ocp.min({ timeIntegral( 1/2*trolley.acceleration^2 ) });
ocp.st(...
    'systems', bdSystem, ...
    ... % Initial conditions
    {  0  '==' t_0( trolley.position ) }, ...
    {  1  '==' t_0( trolley.speed    ) }, ...
    ... % Terminal conditions
    {  1  '==' t_f( time ) }, ...
    {  0  '==' t_f( trolley.position ) }, ...
    { -1  '==' t_f( trolley.speed    ) }, ...
    ... % Constraints
    { 1/9 '>=' trolley.position        } ...
    );

% Solving the OCP
sol = ocp.solve('controlIntervals', 20);

%% Plot the results
figure(1)
subplot(211); hold on
sol.plot(time, trolley.position)
xlabel('Time')
ylabel('Position')

subplot(212); hold on
sol.plot(time, trolley.speed)
xlabel('Time')
ylabel('Velocity')

figure(2); hold on
sol.stairs(time, trolley.acceleration)
xlabel('Time')
ylabel('Acceleration (Control)')


%%
function [dx, y] = trolleyModel(time, state, control)

position = state(1);
speed = state(2);
acceleration = control;
dx = [speed; acceleration];

y.position = position;
y.speed = speed;
y.acceleration = acceleration;

end
```

#### Plots

{% include image.html file="example_images/bryson/brysonDenhamStates.svg" alt="BrysonDenhamControl" caption="Bryson-Denham States" %}

{% include image.html file="example_images/bryson/brysonDenhamControl.svg" alt="BrysonDenhamControl" caption="Bryson-Denham Control" %}


{% include links.html %}