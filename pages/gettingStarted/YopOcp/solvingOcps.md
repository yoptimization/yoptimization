---
title: "Solving optimal control problems"
last_updated: May 5, 2019
keywords: solving optimal control problems, solution, optimize, IPOPT, solve, plot, signal, build, parameterize, symbolic, plotting, discretize, ocp, path, boundary, constraints, re-parametrization, speed-up, satisfied
sidebar: mydoc_sidebar
permalink: solvingOcps
toc: true
folder: gettingStarted/YopOcp
---
To solve optimal control problems Yop uses the (local) direct collocation algorithm. Chapter 11 in Moritz Diehl's text [Numerical Optimal Control](https://www.fs.isy.liu.se/Edu/Courses/NumericalOptimalControl/Diehl_NumOptiCon.pdf){:target="_blank"} gives a good overview of the method.

{% include important.html content="Yop parameterize the control signal as piecewise constant in the discretization. This means that you should plot the control signal using `stairs` to get a correct picture of the solution." %}

## Running `.solve`
To initiate the solution of an optimal control problem it is sufficient to type `ocp.solve`. This will run the direct collocation algorithm with the default options. Below follows a table of available options:

|     Option key       |       Default  value  | Valid value |
|----------------------|-----------------------|-------------|
|`'initialGuess'`      |`ones(size(problem))` |[YopSimulationResults](yopSimulator), [YopOcpResults](solvingOcps#obtaining-the-result), [YopInitialGuess](initialGuess#using-yopinitialguess), empty `[]`, |
|`'controlIntervals'`  |`100`                 |Positive integers |
|`'collocationPoints'` |`'legendre'`          |`'radau'`, `'legendre'` |
|`'polynomialDegree'`  |`5`                   |`1, 2, ..., 9`|
|`'ipopt'`             |`struct`              |[IPOPT options](solvingOcps#ipopt-options) |

Depending on your options, a call to `.solve` can look like:
```matlab
sol = ocp.solve( ...
    'initialGuess', initialGuess, ...
    'controlIntervals', 50, ...
    'collocationPoints', 'legendre', ...
    'polynomialDegree', 3, ...
    'ipopt', struct('max_iter', 5000) ...
    );
```

{% include tip.html content="Only specify the options you want to change from the default ones." %}

{% include tip.html content="Options can be entered in any order." %}

### IPOPT options
To pass options to IPOPT you enter them in a struct. The fieldname of the struct should be the IPOPT option, and the value the option value you wish to set. IPOPT options can be found at [IPOPT options](https://www.coin-or.org/Ipopt/documentation/node40.html){:target="_blank"}.

## How the results are obtained
When running the `.solve` method you obtain a `YopOcpResults` object with the following properties:
```
>> sol = ocp.solve
sol =

  YopOcpResults with properties:

           Variables: [1×1 struct]
    NumericalResults: [1×1 struct]
         NlpSolution: [1×1 struct]
               Stats: [1×1 struct]
```
In the `NumericalResults` property, numerical values of the result are stored as a struct:
```matlab
>> sol.NumericalResults

ans =

  struct with fields:

              Objective: [theOptimalValue]
            Independent: [1×K double]
                  State: [nx×K double]
              Algebraic: []
                Control: [nu×K double]
              Parameter: [np×1 double]
    LagrangeIndependent: [1×K double]
          LagrangeState: [nx×K double]
      LagrangeAlgebraic: []
        LagrangeControl: [nu×K double]
      LagrangeParameter: [np×1 double]
```
You can obtain the results from the struct fields or you can use the built-in symbolic plot functions.

### Symbolic plotting
You can plot the results using the Yop build-in plot functions. These behave like the normal plot functions, except that they take symbolic arguments for the x- and y-axis, instead of numerical values. The following plot methods are available for the `YopOcpResults` class:

| Available plot methods  |
|-----------|
|`.plot(x, y, varargin)`  |
|`.plot3(x, y, z, varargin)` |
|`.stairs(x, y, varargin)`|

The functions are used in the same way as the corresponding Matlab built-in ones, except that you call it like an object method:
```matlab
sol.plot(x, y, varargin)

sol.plot3(x, y, z, varargin)

sol.stairs(x, y, varargin)
```
Plot options such as line width, line color, etc., follow the exact same syntax as that of the regular plot functions. You also specify figure, subplots, and legends in the same way as you would for a normal plot. You also obtain line objects as you would normally:
```matlab
h = sol.plot(x, y, varargin);
```
{% include tip.html content="You can plot any input that can be formed with the [systems variables](yopSystem#yopsystem-variables). It does not have to be a part of the optimzation problem." %}

{% include tip.html content="You can plot a constant value over the problem horizion, type `.plot(t, constantValue)`. Provided you assigned the independent variable to `t`." %}

{% include tip.html content="You can plot any expression that can be formed in [CasADi syntax](https://web.casadi.org/docs/#list-of-operations), for instance `.plot(t, if_else(x > y, x^2, abs(y)))`." %}

{% include important.html content="Yop parameterize the control signal as piecewise constant in the discretization. This means that you should plot the control signal using `stairs` to get a correct picture of the solution." %}

### Signals
Numerical values from an optimization can also be obtained using the `.signals()` method. It behaves in as similar way as the [plot functions](solvingOcps#using-signal), except that it takes only one argument, the expression of the signal you want the numerical value of:
```matlab
value = sol.signal(expression);
```
Also for `.signal` you can input an arbitrary [CasADi expression][CasADi syntax](https://web.casadi.org/docs/#list-of-operations){:target="_blank"} using the [system variables](yopSystem#yopsystem-variables).

## Build, parameterize, optimize
The `.solve` method follows a sequence of first building the symbolic structure of the problem. This parse the symbolic input of the problem formulations and discretize that according to the direct collocation settings. When the build is done, it parameterize the problem. In this step the variables and constraints get their bounds. In the optimize step the problem is handed over to IPOPT that solves (at least tries) it, and the solution is returned in terms of a `YopOcpResults` object. These function can also be run be the user, circumventing the use of `.solve`. This can be useful if you solve the same the same problem formulation, but with different parametrization, as is improves speed a lot not to rebuild the problem.

### Building the discretized OCP's structure
To build the nonlinear program (NLP) structure, resulting from the discretization of the optimal control problem, you run the command `.build`. It takes the following options:

|     Option key       | Default  value | Valid value             |
|----------------------|----------------|-------------------------|
|`'controlIntervals'`  |`100`           |Positive integers        |
|`'collocationPoints'` |`'legendre'`       |`'radau'`, `'legendre'`  |
|`'polynomialDegree'`  |`5`             |`1, 2, ..., 9`           |

Once you have run the command, the build is stored in the `YopOcp` object, it therefore returns no arguments. If you change any of the expressions in the control problem, the command must be rerun, be numerical values can be changed.

### Changing the OCP parametrization after building the structure.
After running the `.build` method, it is still possible to change the parametrization of the problem, that is bounds on the variables.

#### Box constraints
Box constraints by modifying the appropriate properties in the `YopOcp` variable. In the following properties the box constraints are found:
```
>> ocp

ocp =

  YopOcp with properties:

                Independent: [1×1 YopOcpVariable]
                      State: [nx×1 YopOcpVariable]
                    Control: [nu×1 YopOcpVariable]
                  Parameter: [np×1 YopOcpVariable]
```
All properties are `YopOcpVariable`s with the following properties:
```
>> ocp.State

ans =

  2×1 YopOcpVariable array with properties:

    Variable
    Upper
    Lower
    InitialUpper
    InitialLower
    FinalUpper
    FinalLower
    Weight
    Offset
```
As can be seen it is possible to change the bounds, the initial bounds, and the final (terminal) bounds. To do that you use the following syntax
```matlab
% Upper and lower bounds
ocp.State.setUpper(value)
ocp.State.setLower(value)

% Initial bounds
ocp.State.setInitial(value) % sets both upper and lower
ocp.State.setInitialUpper(value)
ocp.State.setInitialLower(value)

% Terminal bounds
ocp.State.setFinal(value)
ocp.State.setFinalUpper(value)
ocp.State.setFinalLower(value)
```
The argument `value` must have dimension as the state variable, unless you specify which variable you set, according to
```matlab
ocp.State(stateNumber).setUpper(value)
ocp.State(stateNumber).setLower(value)
```
You can also specify a range:
```matlab
ocp.State(1:n).setUpper(value)
ocp.State(1:n).setLower(value)
```

#### Path and boundary constraints
To manipulate bounds on path and boundary constraints, the following properties are of interest:
```
>> ocp

ocp =

  YopOcp with properties:

                       Path: [n_path×1 YopPathConstraint]
                 PathStrict: [n_strict×1 YopPathStrictConstraint]
                    Initial: [n_initialx1 YopBoundaryInitialConstraint]
                      Final: [n_finalx1 YopBoundaryFinalConstraint]
```
The path constraints are numbered in the order you provided them. If you are unsure you can inspect them by running `ocp.Path(constraintNumber)`. Every constraint is made up of `YopConstraintComponent`s. These are the scalar entries of the constraints. You can either manipulate these, your you manipulate the constraint directly, it does not matter. To manipulate the boundaries you type:
```matlab
ocp.Path.setUpper(value);
ocp.Path.setLower(value);

% You can also index them
ocp.Path(1:n).setUpper(value);
ocp.Path(1:n).setLower(value);
```
Remember to make sure dimensions are consistent.

#### Parametrization and optimization
Once you have manipulated the variable bounds, you first parameterize the problem. This is done by running `.parameterize`, this will parse your new bounds and it will parameterize the initial guess. Parameterize takes the following options:

|     Option key       |       Default  value  | Valid value |
|----------------------|-----------------------|-------------|
|`'initialGuess'`      |`ones(size(problem))` |[YopSimulationResults](yopSimulator), [YopOcpResults](solvingOcps#obtaining-the-result), [YopInitialGuess](initialGuess#using-yopinitialguess), empty `[]`, |

Once you have parametrize the problem it is solved by running `.optimize`. It takes the following options:

|     Option key       |       Default  value  | Valid value |
|----------------------|-----------------------|-------------|
|`'ipopt'`             |`struct`              |[IPOPT options](solvingOcps#ipopt-options) |

It returns a `YopOcpResults` object.

#### Example of re-parametrization for speed-up
The following code shows how to reparameterize a problem:
```matlab
%% The Bryson-Denham problem
bdSystem = YopSystem(...
    'states', 2, ...
    'controls', 1, ...
    'model', @trolleyModel ...
    );

time = bdSystem.t;
trolley = bdSystem.y;

ocp = YopOcp();
ocp.min({ timeIntegral( 1/2*trolley.acceleration^2 ) });
% ocp.min();
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
    ... % abs makes Yop interpret it as a path constraint
    ... % otherwise it would be a box constraint
    { 1/9 '>=' abs(trolley.position)   } ...
    );

tic
sol(1) = ocp.solve('controlIntervals', 100);
toc

figure(1)
subplot(211); hold on
sol(1).plot(bdSystem.t, bdSystem.x(1))

subplot(212); hold on
sol(1).plot(bdSystem.t, bdSystem.x(2))

figure(2); hold on
sol(1).stairs(bdSystem.t, bdSystem.u)

%% Change initial and final conditions
% ocp.State(1).setInitial(0)
% ocp.State(2).setInitial(2)
ocp.State(1:2).setInitial([0;2])
ocp.State.setFinal([0;-2])
ocp.Path.setUpper(1/5);

tic
ocp.parameterize();
sol(2) = ocp.optimize;
toc

figure(1)
subplot(211); hold on
sol(2).plot(bdSystem.t, bdSystem.x(1))

subplot(212); hold on
sol(2).plot(bdSystem.t, bdSystem.x(2))

figure(2); hold on
sol(2).stairs(bdSystem.t, bdSystem.u)

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

{% include image.html file="people/yoptimalityconditionssatisfied.jpg" alt="Yoptimality conditions satisfied."%}
