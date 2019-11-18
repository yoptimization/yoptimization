---
title: "Animate The Goddard Rocket"
last_updated: June 14, 2018
keywords: animation, goddard, rocket
sidebar: mydoc_sidebar
permalink: goddardAnimate
toc: false
folder: examples/goddardRocket
---

Animation file for Goddard rocket problems, the animation is saved as a gif. Input is the `ocp.solve()` object, filename for the gif, and total time the gif should take. E.g. `animateRocket(ocpSol, 'filename.gif', 6)`.
```matlab
% Author: Dennis Edblom
function animateRocket(ocpSol, filename, time)

t = [];
X = [];
U = [];
for nPhase = 1:length(ocpSol)
    % Get variales
    t = [t; ocpSol(nPhase).NumericalResults.Independent'];
    X = [X; ocpSol(nPhase).NumericalResults.State'];
    U = [U; ocpSol(nPhase).NumericalResults.Control'];
end
% adding an extra frame with
t = [t;t(end)];
X = [X;X(end,:)];
U = [U;0];


% state vectors
h = X(:,2);

nImages = length(X);
% dt = t(2)-t(1);
dt = time/(nImages);


tmin = min(t);
tmax = max(t);
umin = min(U);
umax = max(U);
xmin = min(X);
xmax = max(X);
hmin = min(X(:,2));
hmax = max(X(:,2));
xl = (hmax-hmin)*0.2;
scale = 0.015*(hmax-hmin);
ymax = hmax + scale*7; % Rocket visible at the top
ymin = hmin - scale*4; % Thrust vector visible
nx = 3;
nu = 1;
xlabels = {'v', 'h', 'm'};

% Create the plot
% The first image will be initialized twice which removes some stutter in
% the gif image.
fig = figure('Position',[50 50 1000*0.8 800*0.8]);
set(fig,'color','w');

for k=1:nx
    % Create subplot
    subplot(nx+nu, 2, k*2)
    xlim([tmin,tmax]);
    ylim([xmin(k),xmax(k)]);
    % Add labels
    ylabel(xlabels{k});
    xlabel('Time [t]');

end
% Control
subplot(nx+nu, 2, 8)
xlim([tmin,tmax]);
ylim([umin,umax]);
% Add labels
ylabel('Control F');
xlabel('Time [t]');


subplot(4,2,[1 3 5 7]);
plot([-xl,xl],[hmin hmin],'k','LineWidth',2) %ground
xlim([-xl,xl]); % TODO make scalable with hmax
ylim([ymin,ymax]);
ylabel('Height h');
% first pos
Y = h(1);
T = U(1)/umax;
engine = patch(scale*[0 0 0.5 -0.5], Y + scale*[2 2 0 0], 'b');
body = patch(scale*[-0.5 0.5 0.5 -0.5] , Y + scale*(3 + [2 2 -2 -2]),'b');
nose = patch(scale*[0 0 0.5 -0.5], Y + scale*(5 + [1 1 0 0]), 'b');
exhaust1 = patch(T*scale*[-0.35 0.35 0.35 -0.35] , Y + scale*(T*[0 0 -2 -2]), 'r');
exhaust2 = patch(T*scale*[0.75 -0.75 0 0], Y - scale*(T*2 - T*[0 0 -1 -1]), 'r');



t_text = text(-0.8*xl,hmax,'t = 0');
frame = getframe(fig);
im{1} = frame2im(frame);
% Animation Loop
for i = 1:nImages
    for k=1:nx
        % Create subplot
        subplot(nx+nu, 2, k*2)

        plot(t(1:i), X(1:i,k),'LineWidth',2)
        xlim([tmin,tmax]);
        ylim([xmin(k),xmax(k)]);

        % Add labels
        ylabel(xlabels{k});
        xlabel('Time [t]');

    end
    % Plot control
    subplot(nx+nu, 2, 8)
    stairs(t(1:i), U(1:i),'LineWidth',2)
    xlim([tmin,tmax]);
    ylim([umin,umax]);
    % Add labels
    ylabel('Control F');
    xlabel('Time [t]');

    % Remove old cart
    delete(engine);
    delete(body);
    delete(nose);
    delete(exhaust1);
    delete(exhaust2);
    delete(t_text);

    subplot(4,2,[1 3 5 7]);
    % Draw rocket
    xlim([-xl,xl]);
    ylim([ymin,ymax]);
    ylabel('Height h');


    Y = h(i);
    T = U(i)/umax;
    engine = patch(scale*[0 0 0.5 -0.5], Y + scale*[2 2 0 0], 'b');
    body = patch(scale*[-0.5 0.5 0.5 -0.5] , Y + scale*(3 + [2 2 -2 -2]),'b');
    nose = patch(scale*[0 0 0.5 -0.5], Y + scale*(5 + [1 1 0 0]), 'b');
    exhaust1 = patch(T*scale*[-0.35 0.35 0.35 -0.35] , Y + scale*(T*[0 0 -2 -2]), 'r');
    exhaust2 = patch(T*scale*[0.75 -0.75 0 0], Y - scale*(T*2 - T*[0 0 -1 -1]), 'r');

    t_text = text(-0.8*xl,hmax,sprintf('t = %1.3f',t(i)));
    frame = getframe(fig);
    im{i} = frame2im(frame);
end


for i = 1:nImages
    [A,map] = rgb2ind(im{i},256);
    if i == 1
        imwrite(A,map,filename,'gif','LoopCount',Inf,'DelayTime',0);
        imwrite(A,map,filename,'gif','WriteMode','append','DelayTime',dt); % to remove stutter.
    elseif i == nImages
        imwrite(A,map,filename,'gif','WriteMode','append','DelayTime',1);
    else
        imwrite(A,map,filename,'gif','WriteMode','append','DelayTime',dt);
    end
end
end
```
