---
title: "Double Integrator Simulation Example"
last_updated: June 3, 2019
keywords: example, double integrator, simulation
sidebar: mydoc_sidebar
permalink: doubleIntegratorSim
folder: simulationExamples/doubleIntegrator
toc: false
---

## Code

```matlab
integrator2 = YopSystem('states', 2, 'controls', 1);

integrator2.set('ode', [integrator2.x(2); integrator2.u]);

controller = YopSystem('externals', 2, 'parameters', 2);

u = controller.p(1)*controller.w(1) + controller.p(2)*controller.w(2);

c1 = YopConnection(integrator2.u, u);
c2 = YopConnection(integrator2.x, controller.w);

simulator = YopSimulator(...
    'systems', [integrator2; controller], ...
    'connections', [c1;c2] ...
    );

res = simulator.simulate(...
    'grid', linspace(0, 5, 100), ...
    'initialValue', integrator2.x, [0; 10], ...
    'initialValue', controller.p, [-6; -5] ...
    );

time = integrator2.t;
figure(1);
subplot(311); hold on
res.plot(time, integrator2.x(1));

subplot(312); hold on
res.plot(time, integrator2.x(2));

subplot(313); hold on
res.plot(time, u);
```

### Plot
{% include image.html file="simulation/doubleIntegratorSim.svg" alt="doubleIntegratorSim" caption="Double integrator states and control" %}
