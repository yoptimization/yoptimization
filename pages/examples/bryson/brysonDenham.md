---
title: "Bryson-Denham Problem"
last_updated: April 29, 2019
keywords: Bryson-Denham, Bryson, Denham, example, basics
sidebar: mydoc_sidebar
permalink: brysonDenham
folder: examples/bryson
toc: false
---

The Bryson-Denham problem is a classical minimum energy optimal control
problem and is presented in _Arthur E. Bryson and Yu-Chi Ho. Applied Optimal Control. Taylor Francis
Group, 1975._. The formulation is the following:

$$\min_{a(t)} \frac{1}{2} \int_0^1 a(t)^2 dt$$

$$\dot{v}(t) = a(t) = u(t)$$

$$\dot{x}(t) = v(t)$$

$$v(0)=-v(1)=1$$

$$x(0)=x(1)=0$$

$$x(t) \leq l = \frac{1}{9}$$

where $$u$$ is the control signal.

## Problem visualization
The problem can be visualized as a frictionless box moving sideways with the initial velocity of 1 m/s. The box is only allowed to move a maximum of $$\frac{1}{9}$$ m before it has to turn around. The box has to get back to the starting point after 1 second and it has to have the same velocity when it started but in the other direction. Below is a image visualizing the problem.

{% include image.html file="example_images/bryson/testAnimatedBryson2.gif" alt="BrysonAnimation" caption="Bryson-Denham Problem Visualization" %}

## Solving the problem with Yop
The classic problem is a good starting point for the Yop toolbox. To solve the problem the following steps needs to be implemented. Firstly a model of the problem is needed, the model is usually it's own matlab function file but can be implemented in other ways, see the [YopSystem documentation](yopsystem). Then a YopSystem can be implemented where the number of states and control is specified, see the [YopSystem documentation](yopsystem). After the YopSystem is implemented the optimal control problem can be formulated for the system, see the [YopOcp documentation](optimalcontrol). With the optimal control problem formulated the solution can be found using the [`.solve()`](optimalcontrol#solving-optimal-control-problems) function, then the results can be plotted with the [`.plot()`](optimalcontrol#by-plotting) function.

### Model
Creating the Bryson-Denham model from the [problem formulation](brysonDenham#bryson-denham-problem-formulation).
For more information on how to implement models in Yop see the [YopSystem documentation](yopsystem).

{% include note.html content="Yop assumes the order of the variables in the function handle so be sure that you have the correct order." %}

{% include note.html content="The signal y is created with symbolic vartiables to be used when formulating the OCP." %}

```matlab
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

### Create the YopSystem
Creating a YopSystem with two state variables, one control input and the model [brysonDenhamModel](brysonDenham#model). See [`YopSystem`](yopsystem) for more information.
```matlab
% Create the Yop system
bdSystem = YopSystem(...
    'states', 2, ...
    'controls', 1, ...
    'model', @trolleyModel ...
    );
```

### Formulating the optimal control problem
Setting up the optimal control problem in Yop with [`YopOcp`](optimalcontrol). The cost function and the constraints follow the [problem formulation](brysonDenham#bryson-denham-problem-formulation).

{% include note.html content="Using the signal y from the model to define trolley which is then used for the different conditions." %}

```matlab
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
```

### Solving the optimal control problem
Solving the optimal control problem with the [`.solve()`](optimalcontrol#solving-optimal-control-problems) function.

```matlab
% Solving the OCP
sol = ocp.solve('controlIntervals', 20);
```

### Plotting the results
Plotting the results with the [`.plot()`](optimalcontrol#by-plotting) function.
```matlab
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
```

### The resulting plots
Here are the resulting plots.


{% include image.html file="example_images/bryson/brysonDenhamStates.svg" alt="BrysonDenhamControl" caption="Bryson-Denham States" %}

{% include image.html file="example_images/bryson/brysonDenhamControl.svg" alt="BrysonDenhamControl" caption="Bryson-Denham Control" %}

## Matlab file
Whole code block to easily copy the problem.
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
