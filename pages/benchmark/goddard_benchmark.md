---
title: "Benchmark with goddard rocket final time free"
last_updated: May 5, 2019
keywords: benchmark, goddard, free tf
sidebar: mydoc_sidebar
permalink: goddard_benchmark
folder: benchmark
toc: true
---
This page compares the [Goddard rocket final time free example](goddardRocketFreeTf) between Yop, CasADi and PROPT. The results and the code that generated the optimal solutions can be be found below.

## Optimal solutions

The optimal solutions for the toolboxes are the following:

{% include image.html file="benchmark/states_goddard_diff.svg" alt="states_goddard_diff" caption="State comparison between Yop, CasADi and PROPT." %}

{% include image.html file="benchmark/control_goddard_diff.svg" alt="control_goddard_diff" caption="Control comparison between Yop, CasADi and PROPT." %}

If you look closer you can see that CasAdi and Yop have the same trajectories and PROPT differs a bit.

### Download the comparison
Here is a zip file containing the resulting optimal control data for the different toolboxes and a matlab script plotting the data.

<a target="_blank" class="noCrossRef" href="{{ "yopFiles/goddard_benchmark.zip"}}"><button type="button" class="btn btn-default" aria-label="Left Align"><span class="glyphicon glyphicon-download-alt" aria-hidden="true"></span> Download zip </button></a>

## Code comparison
This section shows the code that generated the optimal solutions for the different toolboxes. Yop and CasADi uses 59 control intervals while PROPT uses 60, this is to make sure all the solutions have the same amount of points in them (60).

### Yop

```matlab
%% Goddard Rocket, Maximum Ascent
% Create the YOP system
sys = YopSystem('states', 3, 'controls', 1, ...
    'model', @goddardRocketModel);
% Symbolic variables
time = sys.t;

% Rocket signals (symbolic)
rocket = sys.y.rocket;

m0 = 214.839;
mf = 67.9833;
Fm = 9.525515;
% Formulate optimal control problem
ocp = YopOcp();
ocp.max({ t_f( rocket.height ) });
ocp.st(...
     'systems', sys, ...
    ... % initial conditions
    { 0  '==' t_0(time)            }, ...
    { 0  '==' t_0(rocket.velocity) }, ...
    { 0  '==' t_0(rocket.height)   }, ...
    { m0 '==' t_0(rocket.mass)     }, ...
    ... % Constraints
    {  0 '<=' t_f( time)          '<=' inf }, ...
    {  0 '<=' rocket.velocity     '<=' inf }, ...
    {  0 '<=' rocket.height       '<=' inf }, ...
    { mf '<=' rocket.mass         '<=' m0  }, ...
    {  0 '<=' rocket.fuelMassFlow '<=' Fm  } ...
    );

% Solving the OCP
sol = ocp.solve('controlIntervals', 59);

%% Plot the results 
figure(1)
subplot(311); hold on
sol.plot(time, rocket.velocity)
xlabel('Time')
ylabel('Velocity')

subplot(312); hold on
sol.plot(time, rocket.height)
xlabel('Time')
ylabel('Height')

subplot(313); hold on
sol.plot(time, rocket.mass)
xlabel('Time')
ylabel('Mass')

figure(2); hold on
sol.stairs(time, rocket.fuelMassFlow)
xlabel('Time')
ylabel('F (Control)')

%% Model
function [dx, y] = goddardRocketModel(t, x, u)
% States and control
v = x(1);
h = x(2);
m = x(3);
F = u;

% Parameters
D0   = 0.01227; 
beta = 0.145e-3;
c    = 2060;    
g0   = 9.81;
r0   = 6.371e6; 

% Drag and gravity
D   = D0*exp(-beta*h);
F_D = sign(v)*D*v^2;
g   = g0*(r0/(r0+h))^2;

% Dynamics
dv = (F*c-F_D)/m-g;
dh = v;
dm = -F;
dx = [dv;dh;dm];

% Signals y
y.rocket.velocity     = v;
y.rocket.height       = h;
y.rocket.mass         = m;
y.rocket.fuelMassFlow = F;
y.drag.coefficient    = D;
y.drag.force          = F_D;
y.gravity             = g;
end
```

### CasADi
```matlab
% Goddard Rocket CasADi implementation.
% Modified from the following example on their homepage.
% https://github.com/casadi/casadi/blob/master/docs/examples/matlab/direct_collocation.m
% https://web.casadi.org/docs/#a-simple-test-problem

import casadi.*

% Degree of interpolating polynomial
d = 3;

% Get collocation points
tau_root = [0 collocation_points(d, 'legendre')];

% Coefficients of the collocation equation
C = zeros(d+1,d+1);

% Coefficients of the continuity equation
D = zeros(d+1, 1);

% Coefficients of the quadrature function
B = zeros(d+1, 1);

% Construct polynomial basis
for j=1:d+1
  % Construct Lagrange polynomials to get the polynomial basis at the collocation point
  coeff = 1;
  for r=1:d+1
    if r ~= j
      coeff = conv(coeff, [1, -tau_root(r)]);
      coeff = coeff / (tau_root(j)-tau_root(r));
    end
  end
  % Evaluate the polynomial at the final time to get the coefficients of the continuity equation
  D(j) = polyval(coeff, 1.0);

  % Evaluate the time derivative of the polynomial at all collocation points to get the coefficients of the continuity equation
  pder = polyder(coeff);
  for r=1:d+1
    C(j,r) = polyval(pder, tau_root(r));
  end

  % Evaluate the integral of the polynomial to get the coefficients of the quadrature function
  pint = polyint(coeff);
  B(j) = polyval(pint, 1.0);
end

% Time horizon
T = MX.sym('T');

% Declare model variables
x1 = SX.sym('x1');
x2 = SX.sym('x2');
x3 = SX.sym('x3');
x = [x1; x2; x3];
u = SX.sym('u');

% Model equations
xdot = rocket_model(x,u);

% Continuous time dynamics
f = Function('f', {x, u}, {xdot});

% Control discretization
N = 59; % number of control intervals
h = T/N;

% Start with an empty NLP
w={}; lbw = []; ubw = []; g={};

% Rocket parameters
m0 = 214.839;
mf = 67.9833;
Fm = 9.525515;

% Final time condition
w = {w{:}, T};
lbw = [lbw; 0];
ubw = [ubw; inf];

% Initial conditions
Xk = MX.sym('X0', 3);
w = {w{:}, Xk};
lbw = [lbw; 0; 0; m0];
ubw = [ubw; 0; 0; m0];

% Formulate the NLP
for k=0:N-1
    % New NLP variable for the control
    Uk = MX.sym(['U_' num2str(k)]);
    w = {w{:}, Uk};
    lbw = [lbw; 0];
    ubw = [ubw;  Fm];

    % State at collocation points
    Xkj = {};
    for j=1:d
        Xkj{j} = MX.sym(['X_' num2str(k) '_' num2str(j)], 3);
        w = {w{:}, Xkj{j}};
        lbw = [lbw;   0;   0; mf];
        ubw = [ubw; inf; inf; m0];
    end

    % Loop over collocation points
    Xk_end = D(1)*Xk;
    for j=1:d
       % Expression for the state derivative at the collocation point
       xp = C(1,j+1)*Xk;
       for r=1:d
           xp = xp + C(r+1,j+1)*Xkj{r};
       end

       % Append collocation equations
       g = {g{:}, h*f(Xkj{j},Uk) - xp};

       % Add contribution to the end state
       Xk_end = Xk_end + D(j+1)*Xkj{j};
    end

    % New NLP variable for state at end of interval
    Xk = MX.sym(['X_' num2str(k+1)], 3);
    w = {w{:}, Xk};
    lbw = [lbw;   0;    0; mf];
    ubw = [ubw; inf;  inf; m0];

    % Add equality constraint
    g = {g{:}, Xk_end-Xk};
end

% Create an NLP solver
prob = struct('f', -Xk(2), 'x', vertcat(w{:}), 'g', vertcat(g{:}));
solver = nlpsol('solver', 'ipopt', prob);

% Solve the NLP
sol = solver('x0', zeros(size(vertcat(w{:}))), 'lbx', lbw, 'ubx', ubw,...
            'lbg', zeros(size(vertcat(g{:}))), ...
            'ubg', zeros(size(vertcat(g{:}))));
w_opt = full(sol.x);

% Plot the solution
T_opt  = w_opt(1);
x1_opt = w_opt(2:4+3*d:end);
x2_opt = w_opt(3:4+3*d:end);
x3_opt = w_opt(4:4+3*d:end);
u_opt  = w_opt(5:4+3*d:end);
tgrid  = linspace(0, T_opt, N+1);

figure(1)
subplot(311); hold on
plot(tgrid, x1_opt)
xlabel('Time'); ylabel('Velocity')

subplot(312); hold on
plot(tgrid, x2_opt)
xlabel('Time'); ylabel('Height')

subplot(313); hold on
plot(tgrid, x3_opt)
xlabel('Time'); ylabel('Mass')

figure(2); hold on
stairs(tgrid, [u_opt; nan])
xlabel('Time'); ylabel('F (Control)')

%% Model
function [dxdt, rocket] = rocket_model(x, u)
% States and control
v = x(1); % Velocity
h = x(2); % Height
m = x(3); % Fuelmass
F = u;    % Thrust

% Parameters
D0   = 0.01227;
beta = 0.145e-3;
c    = 2060;
g0   = 9.81;
r0   = 6.371e6;

% Drag and gravity
D   = D0 * exp( -beta*h );
F_D = sign(v) * D * v^2;
g   = g0 * ( r0 / (r0+h) )^2;

% Dynamics
dv = ( F*c - F_D )/m - g;
dh = v;
dm = -F;
dxdt = [dv;dh;dm];

% Signals y
rocket.velocity = v;
rocket.height = h;
rocket.mass = m;
rocket.fuel_mass_flow = F;
end
```

### PROPT
Taken from the [PROPT manual](https://tomopt.com/docs/propt/tomlab_propt045.php).

```matlab
% PROPT 44
toms t t_f

% Parameters
aalpha = 0.01227; bbeta = 0.145e-3;

c  = 2060;    g0 = 9.81;
r0 = 6.371e6; r02=r0*r0;
m0 = 214.839; mf = 67.9833;
Fm = 9.525515;


for n=[20 40 60]
    
    p = tomPhase('p', t, 0, t_f, n);
    setPhase(p);
    tomStates h v m
    tomControls F
    
    % Initial guess
    if n==20
        x0 = {t_f==250
            icollocate({v == 0; h == 0
            m == m0})
            collocate(F == Fm)};
    else
        x0 = {t_f == tfopt
            icollocate({v == vopt; h == hopt
            m == mopt})
            collocate(F == Fopt)};
    end
    
    % Box constraints
    cbox = {100 <= t_f <= 300
        icollocate({0 <= v; 0 <= h
        mf <= m <= m0
        0 <= F <= Fm})};
    
    % Boundary constraints
    cbnd = {initial({v == 0; h == 0; m == m0})
        final({v==0; m == mf})};
    
    D = aalpha*v.^2.*exp(-bbeta*h);
    g = g0; % or g0*r02./(r0+h).^2;
    
    % ODEs and path constraints
    ceq = collocate({dot(h) == v
        m*dot(v) == F*c-D-g*m
        dot(m) == -F});
    
    % Objective
    objective = -1e-4*final(h);
    
    
    options = struct;
    options.name = 'Goddard Rocket 1';
    options.Prob.SOL.optPar(30) = 30000;
    solution = ezsolve(objective, {cbox, cbnd, ceq}, x0, options);
    
    % Optimal v and more to use as starting guess
    vopt = subs(v, solution);
    hopt = subs(h, solution);
    mopt = subs(m, solution);
    Fopt = subs(F, solution);
    tfopt = subs(t_f, solution);
    
end

t = subs(collocate(t),solution);
v = subs(collocate(vopt),solution);
h = subs(collocate(hopt),solution);
m = subs(collocate(mopt),solution);
F = subs(collocate(Fopt),solution);

figure(1)
subplot(311); hold on
plot(t, v)
xlabel('Time'); ylabel('Velocity')

subplot(312); hold on
plot(t, h)
xlabel('Time'); ylabel('Height')

subplot(313); hold on
plot(t, m)
xlabel('Time'); ylabel('Mass')

figure(2); hold on
stairs(t, F)
xlabel('Time'); ylabel('F (Control)')
```