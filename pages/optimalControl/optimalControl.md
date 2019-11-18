---
title: "Optimal control"
last_updated: April 26, 2019
keywords: Optimal control, optimization, Getting stared, basics
sidebar: mydoc_sidebar
permalink: optimalcontrol
folder: optimalControl/optimalControl
---
## Numerical optimal control basics

### Outline of the direct collocation method

## Formulating optimal control problems
In Yop, optimal control problem are represented by the `YopOcp` class. It can represent optimal control problems on the following form:
```matlab
% An optimal control problem
ocp = YopOcp();

% Objective function to minimize
ocp.min({ t_f( endStateCost ) '+' timeIntegral( integralCost ) })

% Subject to the constraints
ocp.st(...
    'system', mySystem, ...                 % a YopSystem on ODE form
    { t0lb   '<=' t_0(t) '<=' t0ub   }, ... % independent variable initial value bounds
    { tflb   '<=' t_f(t) '<=' tfub   }, ... % independent variable final value bounds
    { x_min  '<='   x    '<=' x_max  }, ... % state bounds
    { x0_min '<=' t_0(x) '<=' x0_max }, ... % state initial value bounds
    { xf_min '<=' t_f(x) '<=' xf_max }, ... % state final value bounds
    { u_min  '<='   u    '<=' u_max  }, ... % control bounds
    { u0_min '<=' t_0(u) '<=' u0_max }, ... % control initial value bounds
    { uf_min '<=' t_f(u) '<=' uf_max }, ... % control final value bounds
    { p_min  '<='   p    '<=' p_max  }, ... % parameter bounds
    { h_min  '<='   h    '<=' h_max  }, ... % path constraints
    { h0_min '<=' t_0(h) '<=' h0_max }, ... % initial boundary constraints
    { hf_min '<=' t_f(h) '<=' hf_max }  ... % final boundary constraints
    );
```
The call to `YopOcp` takes no arguments an returns an optimal control problem object. To formulate an optimal control problem, two methods need to be called. `.min()` and `.st()`. In the call to `.min()` the user sets the objective function to minimize, and in the call to `.st()` the constraints. The boundaries should be specified as column vectors. Also expressions for path constraints should be column vectors.

The call to `.min()` takes one cell array as argument. The cell array either have one or three entries, according to:
```matlab
% Alternative 1
ocp.min({ t_f( endStateCost ) '+' timeIntegral( integralCost ) })

% Alternative 2
ocp.min({ timeIntegral( integralCost ) })

% Alternative 3
ocp.min({ t_f( endStateCost ) })
```
The function `t_f(expression)` indicates to Yop the this expression is to be evaluated at the final boundary. Likewise does the function `t_0(expression)` indicated that the expression is to be evaluated at the initial boundary.

The call to `.st()` always begin with the keyword `'systems'` followed by the system of interest (future releases will support multi-system formulations), the constraints are then specified in arbitrary order using cell arrays. The most general form of providing constraints were presented above, but sometimes there is no upper or lower bound, then the following syntax can be used:
```matlab
% Alternative 1
ocp.st(...
    'system', mySystem, ...
    { h_min '<=' h '<=' h_max } ... % arbitrary constraint
    );

% Alternative 2
ocp.st(...
    'system', mySystem, ...
    { h_min '<=' h }    ... % arbitrary lower bounded constraint
    );

% Alternative 3
ocp.st(...
    'system', mySystem, ...
    { h_max '>=' h }    ... % arbitrary upper bounded constraint
    );

% Alternative 4
ocp.st(...
    'system', mySystem, ...
    { h_fix '==' h }    ... % arbitrary equality constraint
    );

% It is also possible to mix them
ocp.st(...
    'system', mySystem, ...
    { h1_min '<=' h1 '<=' h1_max }, ...
    { h2_min '<=' h2             }, ...
    { h3_max '>=' h3             }, ...
    { h4_fix '==' h4             } ...
    );
```
Concerning the box constraints (bounds on variables):
```matlab
{ t0lb   '<=' t_0(t) '<=' t0ub   } % independent variable initial value bounds
{ tflb   '<=' t_f(t) '<=' tfub   } % independent variable final value bounds
{ x_min  '<='   x    '<=' x_max  } % state bounds
{ x0_min '<=' t_0(x) '<=' x0_max } % state initial value bounds
{ xf_min '<=' t_f(x) '<=' xf_max } % state final value bounds
{ u_min  '<='   u    '<=' u_max  } % control bounds
{ u0_min '<=' t_0(u) '<=' u0_max } % control initial value bounds
{ uf_min '<=' t_f(u) '<=' uf_max } % control final value bounds
{ p_min  '<='   p    '<=' p_max  } % parameter bounds
```
Those that are not used, can be removed.

The independent variable is treated in a slightly different way than the other variables. By omitting the constraint
```matlab
{ t0lb '<=' t_0(t) '<=' t0ub } % independent variable initial value bounds
```
Yop sets the bounds to 0, making the independent variable count from 0. By omitting
```matlab
{ tflb '<=' t_f(t) '<=' tfub } % independent variable final value bounds
```
Yop sets the lower bound to zero and the upper bound to `inf`. For the other box constraints
```matlab
{ x_min  '<='   x    '<=' x_max  } % state bounds
{ x0_min '<=' t_0(x) '<=' x0_max } % state initial value bounds
{ xf_min '<=' t_f(x) '<=' xf_max } % state final value bounds
{ u_min  '<='   u    '<=' u_max  } % control bounds
{ u0_min '<=' t_0(u) '<=' u0_max } % control initial value bounds
{ uf_min '<=' t_f(u) '<=' uf_max } % control final value bounds
{ p_min  '<='   p    '<=' p_max  } % parameter bounds
```
by omitting any of them, you assumes an unconstrained variable.

To solve the problem a call to `.solve()` is made. The only required argument is an initial guess:
```matlab
solution = ocp.solve('initialGuess', myInitialGuess);
```
Once a call is made, Yop first builds the problem, which might take a few minutes depending on problem size. When that is done, the problem is handed over to the NLP solver [IPOPT](https://www.coin-or.org/Ipopt/documentation/). This step may take everything from seconds to hours depending on your computer setup, problem size, settings, and initial guess.

## Providing an initial guess
There are three ways of providing Yop with an initial guess. You either provide a `YopSimulationResults` object (simulation results in which the system subject to the optimal control problem is present), or you provide a `YopOcpResults` object (the solution to another optimal control problem subject to your system), or you provide a guess via a call to `YopInitialGuess` in which case you can provide an arbitrary guess.

### Using a simulation
If you use a simulation as initial guess you simply provide the simulations results variable to Yop:
```matlab
% First simulate yourSystem must be present
simulator = YopSimulator(...
    'systems', [sys1; '...'; yourSystem; '...'; sysN], ...
    'connections', [c1; '...'; cM] ...
    );

% Set initial values and simulation grid
% ...

simulationResults = simulator.simulate();

% Formulate optimal control problem
ocp.min({ 'yourObjectiveFunction' })
ocp.st('systems', yourSystem, ...
       ... % your constraints
      );
% ...

% Solve problem
ocpSolution = ocp.solve('initialGuess', simulationResults);
```

### Using a solution to another optimal control problem
If you have an old or other solution to an optimal control problem in which **the same** instance of your system is present you can use that as an initial guess:
```matlab
ocpSolution = ocp.solve('initialGuess', otherOcpSolution);
```

### Using `YopInitialGuess`
By using `YopInitialGuess` you can provide an initital guess independent of how you obtained it. If your guess is a fixed value for every variable you can write:
```matlab
% Fixed variables (the parameters) are specified in the `parameters` entry,
% the others are specified in `signals`
w0 = YopInitialGuess(...
    'parameters', mySystem.p, ... % The symbolic parameter you want a guess for
    'parameterValues', pValue, ... % The guess value
    'signals', [mySystem.t; mySystem.x; mySystem.u], ... % The symbolic variables you want a guess for
    'signalValues', [tValue; xValue; uValue] ... % The values
    );
```
Every variable of the system subject to the optimal control problem must be accounted for in the initial guess. In the case of a fixed value, Yop will interpret the guess value for the independent variable as a guess for the initial time, and add 1 (tValue + 1) as a guess for the final value. If you have obtained the inital guess via a simulation, the following syntax can be used:
```matlab
w0 = YopInitialGuess(...
    'parameters', mySystem.p, ...
    'parameterValues', pSim, ...
    'signals', [mySystem.t; mySystem.x; mySystem.u], ...
    'signalValues', [tSim; xSim; uSim] ...
    );
```
For `tSim`, `xSim`, `uSim`, the variable dimension grows with columns size, and time grows with rows. For a scalar state variable this means that `xSim` is a row vector. The following figure illustrates the stucture
```
% xSim structure
        |-------------------------------> time
        | x1(t0) ---- x1(t) ---- x1(tf)
        | x2(t0) ---- x2(t) ---- x2(tf)
        |            :
        | xN(t0) ---- xN(t) ---- xN(tf)
        Y
 Number of states
```
**Note** that the grid points must be the same for all signals.

## Solving optimal control problems
As described above, to solve an optimal control problem a cell to `.solve()` specifying an inital guess is sufficient to initiate the solution process:
```matlab
ocpSolution = ocp.solve('initialGuess', simulationResults);
```
However it is possible to specify options. Typically it is of interest to specify the number of control intervals. This is done by writing:
```matlab
sol = ocp.solve('initialGuess', w0, 'controlIntervals', numberOfControlIntervals);
```
It is also possible to specify the collocation points, and the collocation polynomial order. The collocation points are either `legendre` or `radau`. The order is from 1 to 9. The syntax is:
```matlab
ocpSolution = ocp.solve(...
    'initialGuess', simulationResults, ...
    'collocationPoints', 'legendre', ... % or 'radau'
    'polynomialOrder', 5 ...
    );
```
To pass options to IPOPT the following syntax is used:
```matlab
myOptions.ipopt.option = value;
ocpSolution = ocp.solve('initialGuess', simulationResults, 'options', myOptions);
```
IPOPT options can be found at [IPOPT options](https://www.coin-or.org/Ipopt/documentation/node40.html)

## Obtaining the result
When solving a problem
```matlab
ocpSolution = ocp.solve('initialGuess', simulationResults);
```
you obtain an object with the following properties:
```
>> ocpSolution
ocpSolution =

  YopOcpResults with properties:

           Variables: [1×1 struct]
    NumericalResults: [1×1 struct]
         NlpSolution: [1×1 YopNlpSolution]
```
The numerical results are stored in the `NumericalResults` field:
```
>> ocpSolution.NumericalResults

ans =

  struct with fields:

    Independent: [1×(K+1) double]
          State: [nx×(K+1) double]
      Algebraic: [nz×(K+1) double]
        Control: [nu×K double]
      Parameter: [np double]
```
You can obtain the results using this, or you can use the build in methods `.signal(expression)`, `.plot(xExpression, yExpression)` and `.stairs(xExpression, yExpression)`.

### Using `.signal()`
By calling `.signal()` you can obtain the numerical result of any signal/expression in the optimal control problem. For instance to obtain the state of system `mySystem`, you can write:
```matlab
% All state variables
x = ocpSolution.signal(mySystem.x)

% State k
xk = ocpSolution.signal(mySystem.x(k))

% the signal state 1 >= state 2
x1geq2 = ocpSolution.signal( mySystem.x(1) >= mySystem.x(2) )
```

### By plotting
In a similar way to `.signal()` you can plot your results:
```matlab
t = mySystem.t;
x = mySystem.x;
t = mySystem.u;

% States and controls
figure(1)
subplot(311)
ocpSolution.plot(t, x(1))
subplot(312)
ocpSolution.plot(t, x(2))
subplot(313)
ocpSolution.stairs(t, u)

% A phase plane
figure(2)
ocpSolution.plot(x(1), x(2))
```
**Important** When plotting the control signal it is preferable to plot it using `.stairs()`. The reason for this is that the control signal is parameterized as piecewise constant in the optimal control problem. To get a correct picture of the solution you therefore need to plot it using `.stairs()`.

## Validating the results

## Advanced

### Accidental formulation of a path constraint when a box constraint is desired

### Strict path constraints

## Trouble shooting

### When the solver does not converge is does not start iterating
It is not always the case that the problem is solvable. This may have several reasons and there is no general answer. But there are things to try. A simple things is to try and find a feasible solution, which means setting the objective function to zero. This may lead to strange a solution, but if it converge it indicates that the constraints are feasible. Another source of problem is the initial guess, which gives the starting point of the NLP solver. If you have system dynamics that generate imaginary values if they start outside their valid state space, this can cause the solver to refuse starting to iterate.

## Current limitations
Currently Yop only supports ordinary differential equations, that is systems on the form:
```matlab
dx = f(t, x, u, p)
```
Work is in progress for supporting semi-explicit differential algebraic equations (DAEs) of differential index 1. As the target system is a DAE of index 1, it should be a relatively moderate effort to rewrite such a system into an ODE which can the be fed to Yop. **Note** that it is still possible to simulate DAEs of index 1 and use that as an initial guess as long as one of the simulated system is the dynamic constraint in the optimal control problem.
