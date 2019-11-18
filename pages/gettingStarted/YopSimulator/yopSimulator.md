---
title: "Simulation"
last_updated: May 5, 2019
keywords: YopSimulator, Simulation, supported systems, simulating, signal, symbolic, plot
sidebar: mydoc_sidebar
permalink: yopSimulator
toc: true
folder: gettingStarted/YopSimulator
---
## Supported systems
Simulation in Yop is done through the `YopSimulator` class. It takes as input `YopSystem`s and `YopConnection`s and outputs a `YopSimulator` object. Yop simulates semi-explicit differential algebraic equations (DAEs) of differential index-1 using the IDAS integrator from [Sundials](https://computation.llnl.gov/projects/sundials/idas){:target="_blank"}. This means that it can simulate systems on the form:
```matlab
dx = f(t,x,z,p)
0 == g(t,x,z,p)
```
The user must therefore provide `YopSystem`s and `YopConnection`s such that when combined they form the above system.

{% include note.html content="There must be no free control inputs or external inputs (exogenous variables). Connecting your system to a controller system if that is the case, or connect the controller input to a constant value. See [Connections](connections)" %}

## Declaring a `YopSimulator`
To declare a `YopSimulator` the following syntax is used
```matlab
simulator = YopSimulator(...
    'systems', [sys1, '...', sysN], ...
    'connections', [c1, '...', cM] ...
    );
```
Only the `'systems'` argument is necessary, but `'connections'` need to be specified if several systems are provided.
{% include tip.html content="If your systems allow it, you can create several simulators to test different ways of connecting your systems." %}

## Simulating
To simulate you run the command `.simulate`. It requires that you specify an output grid at which points you obtain the numerical value of the simulation. It also requires that you specify initial values for the states and parameters. This is done using the following syntax:
```matlab
res = simulator.simulate(...
    'grid', linspace(t0, tf, samples), ...
    'initialValue', system1.x, x0, ...
    'initialValue', system2.x, 0, ...
    'initialValue', system2.p ...
    );

% You can also write for a nuber of elements
res = simulator.simulate(...
    'grid', linspace(t0, tf, samples), ...
    'initialValue', system1.x(1), x0(1), ...
    'initialValue', system1.x(2), x0(2), ...
    'initialValue', system1.x(3:end), x0(3:end), ...
    'initialValue', system2.x, 0, ...
    'initialValue', system2.p ...
    );
```

{% include tip.html content="A good value for `'grid'` is `linspace(t0, tf, n_samples)`. "%}

### Options
It is also possible to specify options. The follwing options are available.

|   Option key  |  Default  value | Valid value     |
|---------------|-----------------|-----------------|
|`'printStats'` | `false`         | `true`, `false` |
|`'abstol'`     | `1e-8`          | Positive number |
|`'reltol'`     | `1e-6`          | positive number |

Enter the options by typing:
```matlab
res = simulator.simulate(...
    'grid', linspace(t0, tf, samples), ...
    'initialValue', system1.x, x0, ...
    'initialValue', system2.x, 0, ...
    'initialValue', system2.p, ...
    'option', value ...
    );
```

### Obtaining the results
The results are obtained in a `YopSimulationResults` object. The object has the following properties:
```
YopSimulationResults with properties:

  NumericalResults: [1×1 struct]
         Variables: [1×1 struct]
             Stats: [1×1 struct]
```
Numerical results are stored in the numerical results property:
```
struct with fields:

  Independent: [1×ngridpoints double]
        State: [nx×ngridpoints double]
    Algebraic: [nz×ngridpoints double]
    Parameter: [np×ngridpoints double]
```
Variables are stacked on top of each other depending on in which order you entered the systems. To circumvent confusion regarding which variable is which, use can use the `.signal` method to get the numerical results.

#### Numerical results using `.signal`
By calling your results variable with the method `.signal` you can obtain the result of any expression that can be formed using the [system variables](yopSystem#yopsystem-variables). The syntax is the following:
```matlab
sol = simulator.simulate('...');
y = sol.signal(expressionY);
```

#### Numerical results by symbolic plotting
You can plot the results using the Yop build-in plot functions. These behave like the normal plot functions, except that they take symbolic arguments for the x- and y-axis, instead of numerical values. The following plot methods are available for the `YopSimulationResults` class:

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
