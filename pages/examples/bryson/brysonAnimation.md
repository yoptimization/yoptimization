---
title: "Bryson-Denham Animation File"
last_updated: May 6, 2019
keywords: Bryson-Denham, Bryson, Denham, animation, animate
sidebar: mydoc_sidebar
permalink: brysonAnimation
folder: examples/bryson
toc: false
---
Animation file for the Bryson-Denham problem, the animation is saved as a gif. The animation function is called with the solution from the brysonDenham problem and a filename. E.g. `animateBryson(sol,'filename.gif')`.



```matlab
function animateBryson(ocpSol,filename)

t = [];
x_vec = [];
u_vec = [];
for nPhase = 1:length(ocpSol)
    % Get variales
    t = [t; ocpSol(nPhase).NumericalResults.Independent'];
    x_vec = [x_vec; ocpSol(nPhase).NumericalResults.State'];
    u_vec = [u_vec; ocpSol(nPhase).NumericalResults.Control'];
end
dt = t(2)-t(1);

Wrad = 0:.02:2*pi;
Wx = 0.005*cos(Wrad);
Wy = 0.005*sin(Wrad);

nImages = length(x_vec);

tmin = min(t);
tmax = max(t);
umin = min(u_vec)*1.1;
umax = max(u_vec)*1.1;
vmin = min(x_vec(:,2))*1.1;
vmax = max(x_vec(:,2))*1.1;

xmax = max(x_vec(:,1))*1.1;



% Create the plot
% The first image will be initialized twice which removes some stutter in
% the gif image.
h = figure('Position',[50 50 1000*0.8 800*0.8]);
set(h,'color','w');
subplot(2,1,1)
yyaxis left
xlim([tmin,tmax]);
ylim([umin,umax]);
xlabel('Time t');
ylabel('Control signal u');

yyaxis right
xlim([tmin,tmax]);
ylim([vmin,vmax]);
xlabel('Time t');
ylabel('Velocity')

subplot(2,1,2)
plot([-1 2],[0 0],'k','LineWidth',2) %ground
xlim([-0.05,xmax]);
ylim([0,(xmax+0.05)/2.5]);
xlabel('Position');
% first pos
Xcg = x_vec(1,1)-0.02;
Ycg = 0.025;
cart = patch(Xcg+[-0.02 0.02 0.02 -0.02] , Ycg +[0.015 0.015 -0.015 -0.015],'g'); %cart
w1 = patch(Wx+Xcg-0.015, Wy+.005,'r'); % wheel
w2 = patch(Wx+Xcg+0.015, Wy+.005,'r'); % wheel
t_text = text(-0.04,0.08,'t = 0');
frame = getframe(h);
im{1} = frame2im(frame);
% Animation Loop
for i = 1:nImages
    % Plot control signal
    subplot(2,1,1)
    yyaxis left
    stairs(t(1:i),u_vec(1:i),'LineWidth',2)
    xlim([tmin,tmax]);
    ylim([umin,umax]);
    xlabel('Time t');
    ylabel('Control signal u');

    % Plot velocity
    yyaxis right
    plot(t(1:i),x_vec(1:i,2),'LineWidth',2)
    xlim([tmin,tmax]);
    ylim([vmin,vmax]);
    xlabel('Time t');
    ylabel('Velocity')

    % Remove old cart
    delete(cart);
    delete(w1);
    delete(w2);
    delete(t_text);

    % Draw cart
    subplot(2,1,2)
    xlim([-0.05,xmax]);
    ylim([0,(xmax+0.05)/2.5]);
    xlabel('Position');


    Xcg = x_vec(i,1)-0.02;
    Ycg = 0.025;
    cart = patch(Xcg+[-0.02 0.02 0.02 -0.02] , Ycg +[0.015 0.015 -0.015 -0.015],'g'); %cart
    w1 = patch(Wx+Xcg-0.015, Wy+.005,'r'); % wheel
    w2 = patch(Wx+Xcg+0.015, Wy+.005,'r'); % wheel

    t_text = text(-0.04,0.08,sprintf('t = %1.3f',t(i)));
    frame = getframe(h);
    im{i} = frame2im(frame);
end



for i = 1:nImages
    [A,map] = rgb2ind(im{i},256);
    if i == 1
        imwrite(A,map,filename,'gif','LoopCount',Inf,'DelayTime',0);
        imwrite(A,map,filename,'gif','WriteMode','append','DelayTime',dt*10); % to remove stutter.
    elseif i == nImages
        imwrite(A,map,filename,'gif','WriteMode','append','DelayTime',1);
    else
        imwrite(A,map,filename,'gif','WriteMode','append','DelayTime',dt*10);
    end
end
end
```
