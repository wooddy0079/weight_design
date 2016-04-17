# weight_design
For my dissertation
clear
clc
%% Input.
% Initial Condition.
% 效率
Edischarge=0.95;%電池放電效率
Emotor=0.9;%馬達效率
Epropeller=0.8;%螺旋槳效率
% 常數
fdod=0.9;%電池放電深度因子
% Flight Altitude. (m)
h = 100;
[C,a,P,rho,g,mu]=Standard_Atmosphere(h);
% density of wing (Nt/m^2)
Dwing = 1*g;
% 電池物理係數 Panasonic 18650B
Rbattery=243/g;%  (高放電係數) (Wh/Nt) 
Mubattery=0.0485;%  (kg) 
% 飛行時間 
Tcruising=1; % (hr)
% weight. ( N )
W_payload = (10+Mubattery)*g; % One more battery
W_propulsion = 2*g;
W_electronics = 2*g;
% Aspect ratio.
AR = 7;
%
e = 1;
K=1/(pi*e*AR);
%% Initial guess.
flag = 0;
 V_opt = linspace(5,100,200);
% V_opt =30
Sref = 3.0;

% Try Sref
while flag < 1
    
Cd0= 0.045;
cl = 1./sqrt(K./(3.*Cd0));
Wing_loading = rho.* V_opt.^2./(2.*sqrt(K./(3.*Cd0)));
%
temp0 = Tcruising.*1.1.*sqrt(2.*Wing_loading.^3./rho)./...
    (Emotor.*Epropeller.*fdod.*Edischarge.*Rbattery);
temp0 = temp0./(0.25.*(3./(K.*Cd0.^(1/3))).^(3/4));
%
temp1 = Wing_loading - temp0 - Dwing;
Sref_estimate = (W_payload + W_propulsion + W_electronics)./temp1;
%
if (abs(Sref_estimate - Sref) <= 1.0e-4)
    
    
    
    W_total = Wing_loading .* Sref;
    
    flag = 1;
    x = V_opt;
    y1 = W_total;
    y2 = Sref;
    indexmin1 = find(min(y1) == y1);
    indexmin2 = find(min(y2) == y2);
    % at minimum point
    % 座標化，找出最小W_total最小的那點座標
    x1min = x(indexmin1) % V_opt  
    y1min_wTotal = y1(indexmin1) % W_total
    % 座標化，找出最小Wing Area最小的那點座標
    x2min = x(indexmin2) % V_opt  
    y2min_wArea = y2(indexmin2) % Wing Area
%     yy = get(gca,'ylim'); %  ylim = auto let the axis choose the y-axis limits. 
%     hold on
%     plot([range],y) rang r same means thats a point.
%     draw a vertical line at critical point.
%     plot([x1min x1min],yy)
%     hold on
    %% Total weight v.s. Velocity
    subplot(2,2,1);
    plot(V_opt,W_total);
    hold on
    plot([x1min x1min], y1min_wTotal,'x','color','r')
    title('Subplot 1 : Total Weight')
    xlabel('Velocity'),ylabel('Total Weight')
    hold off,grid on
    %% Reference Wing Area v.s. Velocity
    subplot(2,2,2);
    plot(V_opt,Sref);
    hold on
    plot([x2min x2min], y2min_wArea,'x','color','r')
    title('Subplot 2 : Reference Wing Area')
    xlabel('Velocity'),ylabel('Total Weight')
    hold off,grid on
    %% Power Require v.s. Velocity
    subplot(2,2,3);
    Power_required = sqrt(2.*W_total.^3./rho./Sref)./(0.25.*(3./(K.*Cd0.^(1./3))).^(3./4));
    % point out Vopt at power require
    y3 = Power_required;
    y3min_PR = y3(indexmin1) ;
    %
    plot(V_opt,Power_required,'color','b');
    hold on
    plot([x1min x1min], y3min_PR,'o','color','r')
    title('Subplot 3 : Power Require')
    xlabel('Velocity'),ylabel('Power Require')
    hold off,grid on
    %% Number Of Batteries v.s. Velocity
    subplot(2,2,4);
    temp0 = sqrt(2.*Wing_loading.^3./rho)./...
    (Emotor.*Epropeller.*fdod.*Edischarge.*Rbattery);
    temp0 = temp0./(0.25.*(3./(K.*Cd0.^(1/3))).^(3/4));
    Batter_power = temp0 .* Sref .* Rbattery;
    W_battery = temp0 .* Sref;
    Number_battery = W_battery./g./Mubattery;
    % point out Vopt at Number Of Batteries
    y4 = Number_battery;
    y4min_NB = y4(indexmin1) ;
    %
    plot(V_opt,Number_battery,'color','k');
    hold on
    plot([x1min x1min], y4min_NB,'o','color','r')
    title('Subplot 4 : Number Of Batteries')
    xlabel('Velocity'),ylabel('Number Battery')
    hold off,grid on
    %% figure 2  MB & MS
    figure(2)
    % point out Vopt at Mass Of Batteries
    mass_battery = Mubattery.*Number_battery*g; % N
    W_structure = Dwing.*Sref; % N
    %
    y5 = mass_battery;
    y5min_MB = y5(indexmin1) ;
    y6 = W_structure;
    y6min_WS = y6(indexmin1) ;
    %
    subplot(2,1,1)
    plot(V_opt,mass_battery,'color','k');
    hold on
    plot([x1min x1min], y5min_MB,'o','color','r')
    title('Subplot 1 : Mass Of Batteries')
    xlabel('Velocity ( m/s ) '),ylabel('Mass ( N ) ')
    hold off,grid on
     subplot(2,1,2)
     plot(V_opt,W_structure,'color','b');
    hold on
    plot([x1min x1min], y6min_WS,'o','color','r')
    title('Subplot 2 :  Weight Of Structure')
    xlabel('Velocity ( m/s ) '),ylabel('Mass ( N ) ')
    hold off,grid on
    %% figure3
    figure(3)
    mass_battery_structure = mass_battery + W_structure;
    plot(V_opt,mass_battery,'color','k');
    hold on
    plot(V_opt,W_structure,'color','b');
    hold on
    plot(V_opt,mass_battery_structure,'color','r');
    hold off,grid on
    title('The weight whitch will be changed by reference area')
    xlabel('V ( m/s^2 ) '),ylabel('Mass ( N ) ')
    legend('battery weight','structure weight','battery+structure weight')

else 
   Sref = (Sref + Sref_estimate)/2;
end



end
%% output.
Cl=(2.*W_total)./(rho.*(V_opt.^2).*Sref);
Cd=Cd0+K.*Cl.*Cl;
L=0.5.*rho.*V_opt.^2.*Sref.*Cl;
% 
b=(AR.*Sref).^0.2;
L_D=Cl./Cd;
%
