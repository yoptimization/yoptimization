---
title: "Bryson-Denham Pareto Front"
last_updated: May 6, 2019
keywords: Bryson-Denham, Bryson, Denham, example, Pareto, front
sidebar: mydoc_sidebar
permalink: brysonPareto
folder: examples/bryson
toc: false
---
In this example the Pareto front is calculated by solving the [Bryson-Denham problem](brysonDenham) without the distance constraint for different $$t_f \in [0.1 , 2]$$.

## Problem formulation
The formulation is the following:

$$\min_{a(t)} \frac{1}{2} \int_0^{t_f} a(t)^2 dt$$

$$\dot{v}(t) = a(t) = u(t)$$

$$\dot{x}(t) = v(t)$$

$$ t_f \in [0.1 , 2]$$

$$v(0)=-v(1)=1$$

$$x(0)=x(1)=0$$

where $$u$$ is the control signal.

## Yop implementation
Getting the Pareto front is the perfect problem to show how re-parametrization can save time compared to just using `YopSolve` for many iterations of the same problem but with small changes. See [parametrization and optimization](solvingOcps#parametrization-and-optimization) for more information on parametrization. The following example takes the time for 5 iterations when using `YopSolve` and 20 iteration when using re-parametrization to show the time difference of the two methods.

```matlab
%% Pareto optimal solution for Bryson-Denham
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
    {  0  '<=' t_f( time ) }, ...
    {  0  '==' t_f( trolley.position ) }, ...
    { -1  '==' t_f( trolley.speed    ) } ...
    );


%% Getting the pareto otimal solution using YopSolve
% This is just to show how long it would take if the problem was solved
% every time with YopSolve
% Time point vector, with 5 time points
pareto_time_points1 = 0.1:0.2:1;
tic
for i = 1:length(pareto_time_points1)
    ocp.solve('controlIntervals', 40);
end
yopsolve_time = toc


%% Getting the Pareto otimal solution by reparametrization
% Time point vector, with 20 time points
pareto_time_points2 = 0.1:0.1:2;
tic
% Solving the OCP
ocp.build('controlIntervals', 40);
ocp.parameterize;
for i = 1:length(pareto_time_points2)
    ocp.Independent.setFinalUpper(pareto_time_points2(i));
    ocp.parameterize;
    sol(i)= ocp.optimize;
end
reparam_time = toc

% Objective at time points
sols = [sol.NumericalResults];
J = [sols.Objective];

% Maximum distance from starting point at time points
x = [];
for i = 1:length(pareto_time_points2)
    x(i) = max(sols(i).State(1,:));
end

%% Plot Pareto optimal solution by reparametrization
figure(1)
plot(pareto_time_points2,J)
title('Pareto front')
xlabel('Time')
ylabel('Objective')


figure(2)
plot(x,J)
title('Pareto front')
xlabel('Distance')
ylabel('Objective')


%% Model
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

### Plot
Below is the Pareto front for objective to time, and for objective to distance.

{% include image.html file="example_images/bryson/brysonParetoTime.svg" alt="BrysonParetoTime" caption="Bryson-Denham objective to time Pareto front" %}

{% include image.html file="example_images/bryson/brysonParetoDistance.svg" alt="BrysonParetoDistance" caption="Bryson-Denham objective to distance Pareto front" %}
