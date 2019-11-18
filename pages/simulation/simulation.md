---
title: "Simulation"
last_updated: April 25, 2019
keywords: Simulation, YopSimulator, YopSystem, YopConnection
sidebar: mydoc_sidebar
permalink: simulation
folder: simulation/simulation
---
## Basics
Simulation in Yop is done through the `YopSimulator` class. It takes as input `YopSystem`s and `YopConnection`s. Yop simulates semi-explicit differential algebraic equations (DAEs) of differential index-1 using the IDAS integrator from Sundials. This means that it can only simulate systems on the form:
```matlab
dx = f(t,x,z,p)
0 == g(t,x,z,p)
```
The user must therefore provide `YopSystem`s and `YopConnection`s such that when combined they form the above system. **Note** there must be no free control inputs or external inputs (exogenous variables).

### Declaring a YopSimulator
To declare a `YopSimulator` the following syntax is used
```matlab
simulator = YopSimulator('systems', [sys1, '...', sysN], ...
                         'connections', [c1, '...', cM]);
```
Only the `'systems'` argument is necessary, but `'connections'` need to be specified if several systems are provided. Once the simulator is declared, initial values for all states and parameters need to be specified. This is done using the syntax:
```matlab
simulator.setInitialValue(sys.x, initialValue);
simulator.setInitialValue(sys.p, parameterValue);
```
An output grid must also be specified. These are the values at which the solution is obtained, i.e. all timepoints at which you want the solution. This is specified by writing:
```matlab
% The grid choice is arbitrary. Here a linear ones is used
myGrid = linspace(t0, tf, numberOfSamplePoints)
simulator.setGrid(myGrid);
```
We can now simulate the system:
```matlab
simulationResults = simulator.simulate()
```
The results are given in the form of a `YopSimulationResults` object:
```
>> simulationResults

simulationResults =

  YopSimulationResults with properties:

    NumericalResults: [1×1 struct]
           Variables: [1×1 struct]
```
We can either obtain the results from the `NumericalResults` property. The other possibility is to use the methods `.plot(x,y)`, `.stairs(x,y)`, `.signal(x)`. The `.plot()` and `.stairs()` behaves in the same way as the build in ones in MATLAB, with the exception that they take symbolic expressions for the first two arguments. In certain ways this makes them more usable than MATLAB´s build in. For instace, it is possible to plot any system internal signal, a fixed value without giving it a value for every point on grid, or an if-statement using the CasADi function `if_else`. The syntax looks as follows:
```matlab
simulationResults.plot(sys.y.arbitrarySignal1, sys.y.arbitrarySignal1)

% Signal properties as specified in the same way as for the build in function
simulationResults.plot(sys.y.arbitrarySignal1, sys.y.arbitrarySignal1, 'rx-')

figure(1);
subplot(211); hold on; grid on
simulationResults.plot(sys.y.arbitrarySignal1, sys.y.arbitrarySignal1, 'rx-')

subplot(212); hold on; grid on
simulationResults.plot(sys.y.arbitrarySignal3, sys.y.arbitrarySignal4, 'rx-')
```

## Example: State feedback control of a double integrator
The following example the control of a double integrator. The double integrator is given by the system:
```matlab
function [dx, y] = doubleIntegrator(t, x, u)

dx = [x(2); u];

y.position = x(1);
y.speed = x(2);
y.acceleration = u;
end
```
The system is controlled by a controller with a saturated output. The saturation is implemented using the CasADi function `if_else` which is an if-statement. The following shows the implementation:
```matlab
function y = doubleIntegratorController(t, w, p)

x = w(1:2);
L = p(1:2);
saturationValue = p(3);

u = L'*x;
u_sat = if_else(abs(u) > saturationValue, sign(u)*saturationValue, u);

y.systemPosition = w(1);
y.systemSpeed = w(2);
y.controlSignal = u_sat;
end
```
To simulate the system the following code is used:
```matlab
system = YopSystem('states', 2, ...
                   'controls', 1, ...
                   'model', @doubleIntegrator);

controller = YopSystem('externals', 2, ...
                       'parameters', 3, ...
                       'model', @doubleIntegratorController);

% Connecting the system state to the external input of the controller
c1 = YopConnection(system.x, controller.w);

% Connecting the control signal to the control input of the system
c2 = YopConnection(system.u, controller.y.controlSignal);

simulator = YopSimulator('systems', [system; controller], ...
                         'connections', [c1; c2]);

simulator.setInitialValue(system.x, [5, 5]);
simulator.setInitialValue(controller.p, [-6;-5; 5]);
simulator.setGrid(linspace(0, 10, 1000));

res = simulator.simulate();

figure(1)
subplot(221)
res.plot(t, system.y.position, 'LineWidth', 2)
title('Position')
xlabel('Time')
ylabel('Position')
grid minor

subplot(222)
res.plot(t, system.y.speed, 'LineWidth', 2)
title('Speed')
xlabel('Time')
ylabel('Speed')
grid minor

subplot(2,2,[3,4]); hold on
res.plot(t, system.u, 'LineWidth', 2)
res.plot(t, 5, 'r--', 'LineWidth', 2)
res.plot(t, -5, 'r--', 'LineWidth', 2)
title('Control signal')
xlabel('Time')
ylabel('Acceleration')
legend('Control input', 'Control bounds')
grid minor
```
{% include image.html file="simulation/doubleIntegratorSimulation.png" alt="doubleIntegratorSimulation" caption="Simulation results" %}

## Specifying simulation options
