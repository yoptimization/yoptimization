---
title: "Initial Guess For The Goddard Rocket Solution"
last_updated: June 14, 2018
keywords:
sidebar: mydoc_sidebar
permalink: goddardInitialGuessSolution
toc: false
folder: examples/goddardRocket
---
Solution for the initial [Initial Guess For The Goddard Rocket](goddardInitialEx).

## Code
Full code block for easy copying at the bottom of the page.

### Initial guess by simulation
Simulating the rocket with full throttle until the fuel tank is empty as a initial guess.
```matlab
%% Goddard Rocket Initial Guess
% Create the Yop system
goddard = YopSystem('states', 3, 'controls', 1, ...
    'model', @goddardRocketModel);

time = goddard.t;
rocket = goddard.y.rocket;

m0 = 214.839;
mf = 67.9833;
Fm = 9.525515;
%% Initial guess

controller = YopSystem('externals', 2, 'model', @fuelInjectionController);

c1 = YopConnection(controller.y.fuelRateLimit, Fm);
c2 = YopConnection(controller.y.fuelMassAvailable, rocket.mass-mf);
c3 = YopConnection(controller.y.control, rocket.fuelMassFlow);

simulator = YopSimulator(...
    'systems', [goddard; controller], ...
    'connections', [c1; c2; c3] ...
    );

res = simulator.simulate(...
    'grid', linspace(0, 200, 2000), ...
    'abstol', 1e-3, ...
    'initialValue', rocket.velocity, 0, ...
    'initialValue', rocket.height, 0, ...
    'initialValue', rocket.mass, m0 ...
    );

```

### Plotting the initial guess
The plots of the initial guess compared to the optimal solution are available [below](goddardInitialGuess#plots).

```matlab
%% Plot the initial guess
figure(1)
subplot(311); hold on
res.plot(time, rocket.velocity)
xlabel('Time')
ylabel('Velocity')

subplot(312); hold on
res.plot(time, rocket.height)
xlabel('Time')
ylabel('Height')

subplot(313); hold on
res.plot(time, rocket.mass)
xlabel('Time')
ylabel('Mass')

figure(2); hold on
res.stairs(time, rocket.fuelMassFlow)
xlabel('Time')
ylabel('F (Control)')
```


### Creating and solving the optimal control problem

```matlab
% Formulate optimal control problem
ocp = YopOcp();
ocp.max({ t_f( rocket.height ) });
ocp.st(...
     'systems', goddard, ...
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
sol = ocp.solve('controlIntervals', 60, 'initialGuess', res);
```

### Plotting the optimal results
The plots of the initial guess compared to the optimal solution are available [below](goddardInitialGuess#plots).

```matlab
%% Plot the results
% The legend assumes that the initial guess has ben plotted
figure(1)
subplot(311); hold on
sol.plot(time, rocket.velocity)
legend('Initial guess','Optimal Solution')
xlabel('Time')
ylabel('Velocity')

subplot(312); hold on
sol.plot(time, rocket.height)
legend('Initial guess','Optimal Solution')
xlabel('Time')
ylabel('Height')

subplot(313); hold on
sol.plot(time, rocket.mass)
legend('Initial guess','Optimal Solution')
xlabel('Time')
ylabel('Mass')

figure(2); hold on
sol.stairs(time, rocket.fuelMassFlow)
legend('Initial guess','Optimal Solution')
xlabel('Time')
ylabel('F (Control)')
```

### Model
```matlab
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

### Fuel injector controller for the simulation
```matlab
function y = fuelInjectionController(time, externalInput)

fuelRateLimit = externalInput(1);
fuelMassAvailable = externalInput(2);
controlSignal = fuelRateLimit;

% saturateControl
y.control = if_else(fuelMassAvailable <= 0, 0, controlSignal);
y.fuelRateLimit = fuelRateLimit;
y.fuelMassAvailable = fuelMassAvailable;
end
```
## Plots
{% include image.html file="example_images/goddard/GoddardInitialGuessVsOptimalStates.svg" alt="GoddardInitialGuessVsOptimalStates" caption="Goddard rocket initial guess compared to optimal solution, states" %}

{% include image.html file="example_images/goddard/GoddardInitialGuessVsOptimalControl.svg" alt="GoddardInitialGuessVsOptimalControl" caption="Goddard rocket initial guess compared to optimal solution, control" %}


## Full code block for easy copy
```matlab
%% Goddard Rocket Initial Guess
% Create the Yop system
goddard = YopSystem('states', 3, 'controls', 1, ...
    'model', @goddardRocketModel);

time = goddard.t;
rocket = goddard.y.rocket;

m0 = 214.839;
mf = 67.9833;
Fm = 9.525515;
%% Initial guess

controller = YopSystem('externals', 2, 'model', @fuelInjectionController);

c1 = YopConnection(controller.y.fuelRateLimit, Fm);
c2 = YopConnection(controller.y.fuelMassAvailable, rocket.mass-mf);
c3 = YopConnection(controller.y.control, rocket.fuelMassFlow);

simulator = YopSimulator(...
    'systems', [goddard; controller], ...
    'connections', [c1; c2; c3] ...
    );

res = simulator.simulate(...
    'grid', linspace(0, 200, 2000), ...
    'abstol', 1e-3, ...
    'initialValue', rocket.velocity, 0, ...
    'initialValue', rocket.height, 0, ...
    'initialValue', rocket.mass, m0 ...
    );

%% Plot the initial guess
figure(1)
subplot(311); hold on
res.plot(time, rocket.velocity)
xlabel('Time')
ylabel('Velocity')

subplot(312); hold on
res.plot(time, rocket.height)
xlabel('Time')
ylabel('Height')

subplot(313); hold on
res.plot(time, rocket.mass)
xlabel('Time')
ylabel('Mass')

figure(2); hold on
res.stairs(time, rocket.fuelMassFlow)
xlabel('Time')
ylabel('F (Control)')

%% Initial Guess


% Formulate optimal control problem
ocp = YopOcp();
ocp.max({ t_f( rocket.height ) });
ocp.st(...
     'systems', goddard, ...
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
sol = ocp.solve('controlIntervals', 60, 'initialGuess', res);

%% Plot the results
% The legend assumes that the initial guess has ben plotted
figure(1)
subplot(311); hold on
sol.plot(time, rocket.velocity)
legend('Initial guess','Optimal Solution')
xlabel('Time')
ylabel('Velocity')

subplot(312); hold on
sol.plot(time, rocket.height)
legend('Initial guess','Optimal Solution')
xlabel('Time')
ylabel('Height')

subplot(313); hold on
sol.plot(time, rocket.mass)
legend('Initial guess','Optimal Solution')
xlabel('Time')
ylabel('Mass')

figure(2); hold on
sol.stairs(time, rocket.fuelMassFlow)
legend('Initial guess','Optimal Solution')
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


function y = fuelInjectionController(time, externalInput)

fuelRateLimit = externalInput(1);
fuelMassAvailable = externalInput(2);
controlSignal = fuelRateLimit;

% saturateControl
y.control = if_else(fuelMassAvailable <= 0, 0, controlSignal);
y.fuelRateLimit = fuelRateLimit;
y.fuelMassAvailable = fuelMassAvailable;
end

```
