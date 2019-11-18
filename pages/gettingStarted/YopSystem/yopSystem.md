---
title: "YopSystem"
last_updated: May 4, 2019
keywords: YopSystem, declaration, Supported systems, YopSystem variables, model
sidebar: mydoc_sidebar
permalink: yopSystem
toc: false
folder: gettingStarted/YopSystem
---
## Supported systems
A `YopSystem` is a symbolic representation of a model. It handles semi-expicit differential algebraic equations (DAEs) of differential index 1 on the form:
```matlab
dx = f(t, x, z, u, w, p)  % Differential equation
0 == g(t, x, z, u, w, p)  % Algebraic equation
```
where `t` represents the independent variable (typically time), `x` the state variable, `z` the algebraic variable, `u` the control input, `w` external inputs (exogenous variable), and `p` model parameters.

## Declaration
To create a `YopSystem` it is necessary to specify the dimension of the variable vectors. This is done using the following syntax:
```matlab
mySystem = YopSystem(...
    'states', numberOfStates, ...
    'algebraics', numberOfAlgebraics, ...
    'controls', numberOfControlInputs, ...
    'externals', numberOfExternalInputs, ...
    'parameters', numberOfParameters ...
    )
```
If any of dimensions is 0, that key-value pair can be omitted.
{% include note.html content=" The function call is independent of the input ordering. It is only required to enter arguments as key-value pairs." %}

If the model is implemented in a function, and that function has the argument order `[dx, alg, signals] = f(t, x, z, u, w, p)`, this can included in the command which will automatically set the system dynamics:
```matlab
mySystem = YopSystem(...
    'states', numberOfStates, ...
    'algebraics', numberOfAlgebraics, ...
    'controls', numberOfControlInputs, ...
    'externals', numberOfExternalInputs, ...
    'parameters', numberOfParameters, ...
    'model', @myModelFunction ...
    )
```

When running the command the output looks like (provided a model function handle was given):
```matlab
>> mySystem

mySystem =

  YopSystem with properties:

             Independent: [1×1 casadi.MX]
                   State: [nx×1 casadi.MX]
               Algebraic: [nz×1 casadi.MX]
                 Control: [nu×1 casadi.MX]
           ExternalInput: [nw×1 casadi.MX]
               Parameter: [np×1 casadi.MX]
    DifferentialEquation: [nx×1 casadi.MX]
       AlgebraicEquation: [na×1 casadi.MX]
                  Signal: [ny×1 casadi.MX]
                       t: [1×1 casadi.MX]
                       x: [nx×1 casadi.MX]
                       z: [nz×1 casadi.MX]
                       u: [nu×1 casadi.MX]
                       w: [nw×1 casadi.MX]
                       p: [np×1 casadi.MX]
                     ode: [nx×1 casadi.MX]
                      ae: [na×1 casadi.MX]
                       y: [ny×1 casadi.MX]
```
#### YopSystem variables
The shorthand properties `t`, `x`, `z`, `u`, `w`, `p`, `ode`, `ae`, `y` contains the symbolic variables and expressions representing the model, we may refer to these as *the system variables*. The same information is contained in the other entries, these are used internally, but can of course be used by the user, if that is preferred. The shorthand properties can be changed by modifying the file `yopCustomPropertyNames.m`.

{% include note.html content=" You can modify the name of the dynamic properties my changing them in yopCustomPropertyNames.m." %}

### The `'model'` entry
The entry `'model'` in the declaration assumes a function handle on the following form:
```matlab
function [dx, alg, signals] = myModelFunction(t, x, z, u, w, p)
```
`dx` is the evaluation of `dx = f(t, x, z, u, w, p)`, and `alg` the evaluation of `g(t, x, z, u, w, p) `, both of these must be column vectors. `signals` represent any signal specified by the user. Opposite to `dx` and `alg`, this can be of arbitrary type.
{% include tip.html content=" In the case of many internal signals specified in the `signals` output, a `struct` is a good choice to select for the output." %}

### When your system do not require all of the arguments
If your system do not need one or several of the inputs, then that entry can be omitted, except for the independent variable. For instance if your system only takes states and controls as input, the declaration looks as follows:
```matlab
mySystem = YopSystem(...
    'states', numberOfStates, ...
    'controls', numberOfControlInputs, ...
    'model', @myModelFunction ...
    );
```
This means that correspoding outputs should also be removed (i.e. `alg`, as the system lacks an algebraic variable):
```matlab
function [dx, ~] = myModelFunction(t, x, u)
```
where `~` can be replaced by a signals if desired.
{% include important.html content=" The independent variable `t` can not be removed as an input argument, even if it is not used in the function. It is always assumed to come first." %}

### When the model file do not match the argument order
If your model file do not match the specified argument order, there are two alternative ways of specifying the system dynamics. The simplest way is to use an anonymous function to reorder the arguments:
```matlab
mySystem = YopSystem(...
    'states', numberOfStates, ...
    'algebraics', numberOfAlgebraics, ...
    'controls', numberOfControlInputs, ...
    'externals', numberOfExternalInputs, ...
    'parameters', numberOfParameters, ...
    'model' @(t,x,z,u,w,p) myModelFunction('your argument order')
    );
```
The other way of doing it is to omit the `model` entry. Then you use the symbolic variables to obtain expressions, either by using the variables to compute the output expressions, or using them to make a call to a model file:
```matlab
mySystem = YopSystem(...
    'states', numberOfStates, ...
    'algebraics', numberOfAlgebraics, ...
    'controls', numberOfControlInputs, ...
    'externals', numberOfExternalInputs, ...
    'parameters', numberOfParameters ...
    );

% Order arbitrary, but default used for convenience
[dx, alg, signals] = myModelFunction(mySystem.t, mySystem.x, mySystem.z, ...
                                     mySystem.u, mySystem.w, mySystem.p);

% Set model expressions
mySystem.set('ode', dx);
mySystem.set('alg', alg);

% Not necessary, but can be convenient
mySystem.set('y', signals);
```
