---
title: "Formulating optimal control problems"
last_updated: May 4, 2019
keywords: YopOcp, formulating optimal control problems, problem formulation, scaling, scale, constraints, box, path, multi-phase, objective function, timeIntegral, syntax, sol, solution
sidebar: mydoc_sidebar
permalink: formulatingOptimalControlProblems
folder: gettingStarted/YopOcp
toc: true
---
In Yop, optimal control problem are represented by the `YopOcp` class.

## Two key functions: `t_f()` and `t_0()`
Before diving into how optimal control problems are declared in Yop, two functions `t_f(arg)` and `t_0(arg)` need to be introduced. These functions are implemented as:
```matlab
function t = t_0(expression)
t = YopInitialTimepoint(expression);
end
```
and
```matlab
function t = t_f(expression)
t = YopFinalTimepoint(expression);
end
```
As is seen from the implementation the functions are wrapper functions for `YopInitialTimepoint()` and `YopFinalTimepoint()`. `YopInitialTimepoint()` indicates to Yop that the expression given as argument should be evaluated at the initial boundary, likewise indicates `YopFinalTimepoint()` that the expression should be evaluated at the final, or terminal, boundary. As can be see below, these two functions are used frequently to formulate optimal control problems.

The expression given as argument to the functions is any expression that can be formulated using the [YopSystem variables](yopSystem#yopsystem-variables).

{% include tip.html content=" If `t_f` and `t_0` doesn't describe the independent variable well in your optimal control problem, just make your own wrappers for `YopInitialTimepoint()` and `YopFinalTimepoint()`." %}

## Declaring optimal control problems
A `YopOcp` can be declared and represent optimal control problems on the following form:
```matlab
% An optimal control problem
ocp = YopOcp();

% Objective function to minimize
ocp.min({ t_f( endStateCost ) '+' timeIntegral( integralCost ) })

% or if the aim is to maximize the objective:
ocp.max({ t_f( endStateCost ) '+' timeIntegral( integralCost ) })

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

{% include tip.html content=" Only specify the constraints needed to formulate your problem. More info is found under [box constraints](formulatingOptimalControlProblems#box-constraints)" %}

The call to `YopOcp` takes no arguments an returns an optimal control problem object. To formulate an optimal control problem, two methods need to be called. `.min()` (or `.max()`) and `.st()`. In the call to `.min()` the user sets the objective function to minimize (or maximize). In the call to `.st()` the user specifies the constraints the problem is subject to.

### Declaring the objective function
The call to `.min(arg)` and `.max(arg)` takes one argument `arg`. It is a cell array with either 1 or 3 entries. The objective function call should look like:
```matlab
% Alternative 1
ocp.min({ t_f( endStateCost ) '+' timeIntegral( integralCost ) })

% Alternative 2
ocp.min({ timeIntegral( integralCost ) '+' t_f( endStateCost ) })

% Alternative 3
ocp.min({ timeIntegral( integralCost ) })

% Alternative 4
ocp.min({ t_f( endStateCost ) })
```
#### The function `timeIntegral()`
The function `timeIntegral(argument)` is a wrapper for `YopIntegral()` and implemented as:
```matlab
function L = timeIntegral(expression)
L = YopIntegral(expression);
end
```
{% include tip.html content=" Make your own wrapper for `YopIntegral()` should that make your problem formulations cleaner." %}

### Declaring constraints using `.st`
The first argument to `.st()` always begin with the keyword `systems`. This key is followed by the `YopSystem` (see [YopSystem](yopSystem) for more information) that is the subject of the optimal control problem. This is then followed by the constraints, independent of ordering.

#### Systems that can be used in optimal control problems
Currently Yop only accepts one system in the optimization problem formulation. This system is limited to an ODE (work is in progress for semi-explicit index 1 DAEs, which also enables a multi-system formulation). This means that the system must be on the form

$$ \dot{x} = f(t, x, u, p) $$

If you are not sure whether this applies to your system, you can evaluate whether *you* can implement your system in Simulink. If this is the case, you have an ODE. However, this should not be a big limitation as you can with moderate effort convert your semi-explicit index 1 DAE into an ODE. If it is simply a multi-system formulation you wish to formulate, implement all systems in a single YopSystem.

{% include important.html content=" Only one system, described by an ordinary differential equation, can be used in the optimal control problem. This is a limitation that will be improved as soon as possible." %}

#### Constraints
Constraints are declared (formulated) using cell arrays. The cell array either has 3 or 5 entries. The following are correct implementations of constraints:
```matlab
% Upper and lower bounded
{ lb  '<=' expression '<=' ub }
{ lb  '<=' t_0( expression ) '<=' ub }
{ lb  '<=' t_f( expression ) '<=' ub }

% Lower bounded
{ lb  '<=' expression }
{ lb  '<=' t_0( expression ) }
{ lb  '<=' t_f( expression ) }

% Upper bounded
{ ub  '>=' expression }
{ ub  '>=' t_0( expression ) }
{ ub  '>=' t_f( expression ) }

% Equality constraint
{ eq  '==' expression }
{ eq  '==' t_0( expression ) }
{ eq  '==' t_f( expression ) }
```
The entries `eq`, `lb` and `ub` should have numerical values and can be vector valued. In the case that they are vector valued their dimension must be the same as that of `expression`. `expression` is any expression that can be formed using the [symbolic variables](yopSystem#yopsystem-variables). In case that `eq`, `lb` or `ub` are scalar valued and `expression` is vector valued, Yop assigns the same bound to every element of `expression`.

{% include tip.html content=" If you want the same bound for every element in a vector valued `expression`, set the bound (`eq`, `lb` or `ub`) to that scalar." %}

{% include tip.html content=" This is an equality constraint: `{ eq '<=' expression '<=' eq }`." %}

#### Box constraints
Box constraints are constraints on a single (vector valued) variable. Yop automatically detects which constraints are box constraints, and which are path constraints. Provdid the variable `v` in the code is either the independent variable, state variable, algebraic variable, control variable, or parameter variable, the following constraints will be interpreted as box constraints:
```matlab
% Upper and lower bounded
{ lb  '<=' v '<=' ub }
{ lb  '<=' t_0( v ) '<=' ub }
{ lb  '<=' t_f( v ) '<=' ub }

% Lower bounded
{ lb  '<=' v }
{ lb  '<=' t_0( v ) }
{ lb  '<=' t_f( v ) }

% Upper bounded
{ ub  '>=' v }
{ ub  '>=' t_0( v ) }
{ ub  '>=' t_f( v ) }

% Equality constraint
{ eq  '==' v }
{ eq  '==' t_0( v ) }
{ eq  '==' t_f( v ) }
```

{% include warning.html content=" Only in one way is it possible to make Yop interpret what should be a box constraint as a general path constraint. This happens when you select certain elements in a vector valued variable using the following syntax: `v(1:n)`. If you wish this to be interpreted as a box constraint you need to stack the elements using the syntax `[v(1); '...'; v(n)]`." %}

##### Default box constraints
Omitting to set a box constraint on a variable is an indication to Yop that this variable is unbounded. However, by entering the following constraint:
```matlab
{ lb  '<=' v '<=' ub }
```
Yop will automatically set the same bounds for the initial and final value of that variable. If you want other bounds at the initial or final time you need to enter the constraints:
```matlab
{ lb0  '<=' t_0( v ) '<=' ub0 }
{ lbf  '<=' t_f( v ) '<=' ubf }
```
Or if you wish them to be unbounded write:
```matlab
{ -inf '<=' t_0( v ) '<=' inf }
{ -inf '<=' t_f( v ) '<=' inf }
```

Box constraints on the [independent variable](independent) are treated in a different way, and this is the only exception. By omitting to set an initial bound, Yop will interpret that as starting from zero. That is the constraint:
```matlab
{ 0  '==' t_0( independentVariable ) }
```
that is, to begin counting from 0. By omtting to set a final bound, Yop interpret that as free end time with lower bound. That is:
```matlab
{ 0  '<=' t_f( independentVariable )  '<=' inf }
```


#### Strict path constraints
To avoid creating a nonlinear optimization problem, resulting from the discretization of the optimal control problem, with too few degrees of freedom, (more constraints than free variables) Yop does not impose the path constraints at every discretization point. The default setting is to only impose the path constraints at the beginning of a control interval. Typically this is sufficient and therefore preferable to imposing the constraint at every discretization point. However, this is not always the case. If it is obvious or you suspect that this weakness is exploited by the solver, the function `strict(constraint)` can be used. This tells Yop that this constraint should be imposed at every discretization point. This is not to be used for [box constraints](formulatingOptimalControlProblems#box-constraints) as these are always imposed at every discretization point. An optimal control problem formulation including strict path constraints can look like:
```matlab
% Subject to the constraints
ocp.st(...
    'system', mySystem, ...                      % a YopSystem on ODE form
    { t0lb   '<=' t_0(t) '<=' t0ub   }, ...      % independent variable initial value bounds
    { tflb   '<=' t_f(t) '<=' tfub   }, ...      % independent variable final value bounds
    { x_min  '<='   x    '<=' x_max  }, ...      % state bounds
    { x0_min '<=' t_0(x) '<=' x0_max }, ...      % state initial value bounds
    { xf_min '<=' t_f(x) '<=' xf_max }, ...      % state final value bounds
    { u_min  '<='   u    '<=' u_max  }, ...      % control bounds
    { u0_min '<=' t_0(u) '<=' u0_max }, ...      % control initial value bounds
    { uf_min '<=' t_f(u) '<=' uf_max }, ...      % control final value bounds
    { p_min  '<='   p    '<=' p_max  }, ...      % parameter bounds
    { h_min  '<='   h    '<=' h_max  }, ...      % path constraints
 strict({ h_min  '<='   h    '<=' h_max  }), ... % strict path constraints
    { h0_min '<=' t_0(h) '<=' h0_max }, ...      % initial boundary constraints
    { hf_min '<=' t_f(h) '<=' hf_max } ...       % final boundary constraints
    );
```

{% include tip.html content=" Use `strict()` in the problem formulation when the solver violates a path constraint too much." %}

{% include note.html content=" `strict()` is not meaningful for box constraints as the box constraints are already imposed at every discretization point." %}


### Scaling
Scaling of an optimal control problem can be important for convergence. In Yop you can scale the variables, and objective function. The constraints you need to scale yourself, which can be done using simple rules of thumb. Before going further it should be noted that there is no clear definition of what good scaling is, but certain rules of thumb exist. Typically you want the variable to take values ranging from 0 to 1, or some say -1/2 to 1/2 (personally I think if this choice hampers converge of your problem you probably have bigger issues and should look to other solutions). In Yop scaling of variables is based on the equation:

$$ v_{s} = W v - s $$

where $$v_{s}$$ is the scaled variable, $$W$$ the scale weight, and $$s$$ the shift. For the state variable, we consequently also get the scaling of the dynamics

$$ \dot{x}_{s} = W \dot{x} $$

The objective function is only normalized:

$$ W \Big(  E(t_f, x(t_f), u(t_f), p) + \int_{t_0}^{t_f} L(t, x(t), u(t), p)dt \Big) $$

For path constraint the following scaling can be used:

$$ 0 = W c(t, x, u, p) $$

$$ W c_{lb} \leq W c(t, x, u, p) \leq W c_{ub} $$

{% include note.html content=" Path constraints are scaled by the user in the expression." %}

#### Syntax
For the objective function use the following syntax:
```matlab
ocp.scale('objective', weight);
```
For the variables the following is used:
```matlab
ocp.scale('variable', v, 'weight', w, 'shift', s)
```
{% include note.html content=" When scaling variables, dimensions must agree. Also note that variables can be stacked into a vector." %}


## Multi-phase problems
Certain optimal control problems are best formulated as multi-phase problems, see for instance the [Goddard rocket problem with landing](goddardLanding). In this problem the aim is to maximize height with the constraint of landing the problem. As the end time is free we wish to maximize the height of the problem in the first phase and land the problem in the second phase. Alternatively, we might wish to formulate a problem in which the system dynamics change in a non continuous way, that this change is naturally formulated as a change of phase.

The declaration of a multi-phase problem follows that of a single-phase problem. Begin by declaring an ocp for every phase using the following syntax:
```matlab
ocp(1) = YopOcp;
'...'
ocp(numberOfPhases) = YopOcp;
```
Precede as you would for a single-phase problem but index which problem you are formulating:
```matlab
ocp(n).min()
ocp(n).st()
```
When this is done solve them by running:
```matlab
sol = ocp.solve;
```
The solution object `sol` will have as many entries as you have phases.
{% include note.html content=" Scaling must be done separately for each phase, but scale the variables in the same way." %}
{% include note.html content=" The results (`sol`) are given for each phase respectively." %}
{% include tip.html content=" If the phase problem formulation is meaningful you can solve each phase separately, simply type `ocp(n).solve`" %}
{% include warning.html content=" It is possible to instantiate all phases using only `ocp(numberOfPhases) = YopOcp`, however if you make changes to your problem the old phases numbered $$1..(n_{phases}-1)$$ might store old formulations."%}
