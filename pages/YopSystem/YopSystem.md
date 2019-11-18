---
title: "YopSystem"
last_updated: April 25, 2019
keywords: YopSystem, Getting stared, basics, declaration, symbolic, variables
sidebar: mydoc_sidebar
permalink: yopsystem
folder: YopSystem/YopSystem
---
## Basics
A `YopSystem` is a symbolic representation of a model. It handles semi-expicit differential algebraic equations (DAEs) of differential index 1, on the form:
```matlab
dx = f(t, x, z, u, w, p)  % Differential equation
0 == g(t, x, z, u, w, p)  % Algebraic equation
```
where `t` represents the independent variable (typically time), `x` the state variable, `z` the algebraic variable, `u` the control input, `w` external inputs (exogenous variable), and `p` model parameters.

To declare a `YopSystem`, the following syntax is used:
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
**Note** The function call is independent of the input ordering. The output looks like (variable dimensions vary depending on the input):
```matlab
>> mySystem

mySystem =

  YopSystem with properties:

             Independent: [1×1 casadi.MX]
                   State: [2×1 casadi.MX]
               Algebraic: [1×1 casadi.MX]
                 Control: [3×1 casadi.MX]
           ExternalInput: [2×1 casadi.MX]
               Parameter: [1×1 casadi.MX]
    DifferentialEquation: [2×1 casadi.MX]
       AlgebraicEquation: [1×1 casadi.MX]
                  Signal: [1×1 casadi.MX]
                       t: [1×1 casadi.MX]
                       x: [2×1 casadi.MX]
                       z: [1×1 casadi.MX]
                       u: [3×1 casadi.MX]
                       w: [2×1 casadi.MX]
                       p: [1×1 casadi.MX]
                     ode: [2×1 casadi.MX]
                      ae: [1×1 casadi.MX]
                       y: [1×1 casadi.MX]
```
The properties `t`, `x`, `z`, `u`, `w`, `p`, `ode`, `ae`, `y` contains the symbolic variables and expressions representing the model. The same information is contained in the other entries, these are used internally, but can of course be used by the user, if that is preferred.

The entry `model` assumes a function handle on the following form:
```matlab
function [dx, alg, signals] = myModelFunction(t, x, z, u, w, p)
```
`dx` is the evaluation of `dx = f(t, x, z, u, w, p)`, and `alg` the evaluation of `g(t, x, z, u, w, p) `, both of these must be column vectors. `signals` represent any signal specified by the user. Opposite to `dx` and `alg`, this can be of arbitrary type, in the case of many signals, a `struct` is a good choice.

## The independent variable
The independent variable in Yop is treated in a special way using the class `YopIndependentVariable`. This way all `YopSystem`s share the same independent variable. This is of practical value to the user when plotting or obtaining numerical results from a simulation or optimal control problem, since it means that the independent variable of any `YopSystem` can be used.

## When your system do not require all of the inputs or outputs
If your system happen to not need one or several of the inputs, then that entry can be omitted. For instance if your system only takes states and controls as input, the declaration looks as follows:
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

## When the model file do not match the input-output pattern
If your model file do not match the specified input-output pattern, there are two alternative ways of specifying the system dynamics. The simplest way is to use an anonymous function:
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

## Connecting systems
One of the main reasons for supporting semi-explicit DAEs of differential index 1, is the possibility of connecting ordinary differential equations (ODEs). If the optimal control problem we are interested in solving is non-linear and non-convex, it may be necessary to provide a good initial guess. One way of doing that is by simulation. As the system may be unstable, or for any other reason need a controller in order to be make a simulation, it can be convenient to simply connect two ODEs via an algebraic equation. In Yop systems are connected in the following way:
```matlab
mySystem = YopSystem('states', nx_sys, 'controls', nu, 'model' @myModelFunction);
myController = YopSystem('states', nx_ctrl, 'externals', nx_sys, 'model', @myController);

% Connecting the system state (could be any signal)
% to the controller external input
c1 = YopConnection(mySystem.x, myController.w);

% Connecting the control signal from the myController
% (stored in the signals output as a struct) to the
% control input of the system
c2 = YopConnection(mySystem.u, myController.y.controlSignal);
```
When used in a simulation or optimal control problem, the connections, in this case `c1` and `c2`, need to be specified. **Note** it is possible to define expressions not contained in the model files when connecting the system. For instance if the controller should track a reference value, this can be formulated as the connection:
```matlab
c1 = YopConnection(referenceValue - mySystem.x, myController.w);
```
Or if we want an unconnected signal, let's say the third control input, to remain fixed, we can write:
```matlab
c =  YopConnection(mySystem.u(3), 0);
```

## Important limitations
Yop uses CasADi for symbolic representation of model objects. This means the expressions making up a `YopSystem` can only contain expressions the can be formulated in CasADi's symbolic framework. For instance the if-statesments like:
```matlab
function dx = mySystem(t, x)

% ...

if x > 10
  % ...
  % Some code
  % ...
end

% ...

end
```
In this case it is possible to replace the if-statement with the CasADi function `if_else(expression, trueCase, falseCase)`. Most operations are available in CasADi, but for a complete list, visit the [CasADi](https://web.casadi.org/docs/#list-of-operations) website.
