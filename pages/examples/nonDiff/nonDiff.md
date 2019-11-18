---
title: "Nondifferentiable system"
last_updated: June 3, 2019
keywords: example, nondifferentiable, if_else
sidebar: mydoc_sidebar
permalink: nonDiff
folder: examples/nonDiff
toc: false
---


This problem is taken from the [PROPT manual, example 72.](https://tomopt.com/docs/propt/tomlab_propt073.php){:target="_blank"}

$$J = x_3(t_f)$$

$$\dot{x_1}(t) = x_2$$

$$\dot{x_2}(t) = -x_1-x_2+u+d$$

$$\dot{x_3}(t) = 5 x_1^2 + 2.5 x_2^2 + 0.5 u^2$$

$$d = 100 \cdot [U(t-0.5)-U(t-0.6)]$$

Where U is the unit step function so that $$U(t-\alpha) = 0$$ when $$t < \alpha$$ and $$U(t-\alpha) = 0$$ when $$t > \alpha$$.

$$x_1(0)=0$$

$$x_2(0)=0$$

$$x_3(0)=0$$

## Solving the problem
This problem introduces how to solve nondifferentiable systems. The problem can be solved by using phases where the model is altered when the unit steps get triggered.
The problem can also be solved by using the casadi function **if_else** and thus modeling the unit step as well. By doing this only one phase is needed.


### Main file
```matlab
%% Nondifferentiable system
% Author: Dennis Edblom
sys = YopSystem(...
    'states', 3, ...
    'controls', 1, ...
    'model', @nonDiffModel);
% Symbolic variables
t = sys.t;
x = sys.x;
u = sys.u;

% Phase description
ocp = YopOcp();
ocp.min({ t_f( x(3) ) });
ocp.st(...
     'systems', sys, ...
     ... % Initial conditions
    { 0 '==' t_0(x(1)) }, ...
    { 0 '==' t_0(x(2)) }, ...
    { 0 '==' t_0(x(3)) }, ...
    ... % Final conditions
    { 2 '==' t_f( t )  } ...
    );

w0 = YopInitialGuess(...
    'signals', [t; x; u], ...
    'signalValues', [0; 0; 0; 0; -5] ...
    );

sol = ocp.solve(...
    'controlIntervals', 100, ...
    'collocationPoints', 'radau', ...
    'polynomialDegree', 3, ...
    'initialGuess', w0);

%% Plot the results
figure(1)
subplot(2,1,1)
sol.plot(t,x);
legend('x1','x2','x3');
title('Nondiff System state variable');

subplot(2,1,2)
sol.plot(t,u);
legend('u');
title('Nondiff System control');

%% Model
function dx = nonDiffModel(t, x, u)
% Nondifferentiable part
d = 100*(if_else(t > 0.5,1,0)-if_else(t > 0.6,1,0));

dx1 = x(2);
dx2 = -x(1)-x(2)+u+d;
dx3 = 5*x(1)^2+2.5*x(2)^2+0.5*u^2;
dx = [dx1;dx2;dx3];
end
```
{% include note.html content=" The model uses the CasADi function `if_else` to model $$d = 100 \cdot [U(t-0.5)-U(t-0.6)]$$." %}


## Plot
{% include image.html file="example_images/nonDiff/nonDiffPlot.svg" alt="nonDiffPlot" caption="Nondifferentiable system states and control" %}
