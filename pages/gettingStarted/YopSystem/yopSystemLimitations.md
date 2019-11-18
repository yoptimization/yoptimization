---
title: "Modeling limitations"
last_updated: May 4, 2019
keywords: Getting stared, basics, YopSystem, system, model, limitations
sidebar: mydoc_sidebar
permalink: yopSystemLimitations
toc: false
folder: gettingStarted/YopSystem
---
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
In this case it is possible to replace the if-statement with the CasADi function `if_else(expression, trueCase, falseCase)`. Most operations are available in CasADi, but for a complete list, visit the [CasADi website](https://web.casadi.org/docs/#list-of-operations){:target="_blank"}.

However, it is possible for the function to contain if-statements as long as the conditions do not operate on symbolic variables. The following is possible:
```matlab
function dx = mySystem(t, x, aVariableNotPartOfTheYopSystem)

% ...

if aVariableNotPartOfTheYopSystem > 10
  dx = yourSymbolicExpression;

else
  dx = yourOtherSymbolicExpression;

end

% ...

end
```
