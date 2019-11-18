---
title: "Providing an initial guess"
last_updated: May 3, 2019
keywords: YopOcp, initial guess, simulation, YopInitialGuess, fixed guess value
sidebar: mydoc_sidebar
permalink: initialGuess
toc: true
folder: gettingStarted/YopOcp
---
Much of the time an initial guess is required to solve an optimal control problem. An initial guess is a starting point for the optimization solver to start iterating from. A good start can make the problem converge in fewer iterations, while a poor one can make it crash in the first iteration.

There are four ways of providing an initial guess in Yop. The first and simplest is to omit providing one. This sets the initial guess to ones. The reason it is not zero is an attempt to avoid division by 0 at the starting point. The second way is to provide simulations results. This ensures that the system dynamics are fulfilled at the starting point. The third way is to provide results from another optimal control problem in which the system you optimize is present. The fourth way is to use the general class `YopInitialGuess` to provide an arbitrary guess.

## Using simulation
Simulation is a useful tool for providing an initial guess. This ensures that your initial guess fulfills the system constraints. This is why [simulation](yopSimulator) is a part of Yop. Using `YopSimulator` you can connect your system to a controller in order to simulate the initial guess. When you have simulated you enter the simulations results variable to the `.solve` method. The general syntax look like:
```matlab
% First simulate.
% 'yourSystem' must be present
simulator = YopSimulator(...
    'systems', [sys_1; '...'; yourSystem; '...'; sys_N], ...
    'connections', [c_1; '...'; c_M] ...
    );

simulationResults = simulator.simulate(...
    'grid', linspace(0, 1, 100), ... % Just an example output grid
     ... % Specify initial conditions
    'initialValue', sys_N.x, stateInitialValue ...
    );

% Formulate optimal control problem
ocp.min({ 'yourObjectiveFunction' })
ocp.st('systems', yourSystem, ...
       ... % your constraints
      );
% ...

% Solve problem
ocpSolution = ocp.solve('initialGuess', simulationResults);
```
{% include important.html content=" Yop uses the CasADi symbolic engine to extract the relevant information from the simulated initial guess. It is therefore important that the system provided to the simulation and optimal control problem is the **exact same** instance, otherwise it cannot match the variables. Circumvent this issue by not rerunning the `YopSystem` command every time you run your code." %}

## Using a solution to another optimal control problem
If you have an old or other solution to an optimal control problem in which the same instance of your system is present you can use that as an initial guess. Type:
```matlab
ocpSolution = ocp.solve('initialGuess', otherOcpSolution);
```
{% include important.html content=" Yop uses the CasADi symbolic engine to extract the relevant information from the other solution. It is therefore important that the system provided to the old solution and optimal control problem is the **exact same** instance.  Circumvent this issue by not rerunning the `YopSystem` command every time you run your code." %}

## Using `YopInitialGuess`
By using `YopInitialGuess` you can provide an initial guess independent of how you obtained it. For instance you may hade simulated one using any of Matlab's integrators, or you wish to enter fixed values other than ones. `YopInitialGuess` accepts four keywords: `'parameters'`, `'parameterValues'`, `signals`, and `'signalValues'`. After `parameters` you specify the parameters present in your system. After `'parameterValues'` you specify the initial guess for the parameters. The same goes for signals. After the `signals` you enter the symbolic variables. If the dimension of any of the variables is zero, these can be omitted. After `'signalValues'` you enter the signal initial values in the same order as you entered the variables. The grid points for the intial guess is specified as the values for the independent variable. See [below](initialGuess#dimensions-for-the-guess-values) for the dimensions. The syntax is:
```matlab
w0 = YopInitialGuess(...
    'parameters', mySystem.p, ... % Parameters
    'parameterValues', p0, ... % Guess values
    'signals', [mySystem.t; mySystem.x; mySystem.u], ... % Time-varying variables
    'signalValues', [t0; x0; u0] ... % Guess values
    );
```
{% include important.html content="The grid points are specified in the `t0` entry and these must then be the same for the other variables." %}

{% include warning.html content="If you use `YopInitialGuess` you need to account for all variables `t`, `x`, `u`, and `p` with dimension larger or equal to one." %}

{% include tip.html content="If you do not have any parameters, you do not have to enter `'parameters'` and `'parameterValues'`." %}

### Dimensions for the guess values
For `t0`, `x0`, `u0`, the variable dimension grows with row size, and time grows with columns. For a scalar state variable this means that `x0` is a row vector. For the general case the dimension is $$ \text{dim}(v) \times n_{samples}$$. The following figure illustrates the general structure of an entry:
```
x0 structure
        |-------------------------------> time (columns)
        | x1(t0) ---- x1(t) ---- x1(tf)
        | x2(t0) ---- x2(t) ---- x2(tf)
        |            :
        | xN(t0) ---- xN(t) ---- xN(tf)
        Y
Number of states (rows)
```
{% include warning.html content="If your guess is not a fixed value, you need to have the correct dimensions for the values in your initial guess." %}

### Fixed guess value
If the initial guess you wish to provide is a fixed value. The following syntax is used
```matlab
w0 = YopInitialGuess(...
    'parameters', p, ... % Parameters
    'parameterValues', pValue, ... % Guess values
    'signals', [mySystem.t; mySystem.x; mySystem.u], ... % Time-varying variables
    'signalValues', [tValue; xValue; uValue] ... % Guess values
    );
```
Every variable of the system subject to the optimal control problem must be accounted for in the initial guess. In the case of a fixed value, Yop will interpret the guess value for the independent variable as a guess for the initial time, and add 1 (tValue + 1) as a guess for the final value. If you have obtained the inital guess via a simulation, the following syntax can be used:
```matlab
w0 = YopInitialGuess(...
    'parameters', mySystem.p, ... % Parameters
    'parameterValues', p0, ... % Guess values
    'signals', [mySystem.t; mySystem.x; mySystem.u], ... % Time-varying variables
    'signalValues', [t0; x0; u0] ...
    );
```
When you enter a fixed valued as a guess for the independent variable Yop takes this a guess for the starting point of the independent variable and adds to that 1 to get the end time.
{% include note.html content="`[t0; x0; u0]` is a column vector if your guess is a fixed value." %}
