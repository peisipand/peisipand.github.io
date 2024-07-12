---
layout:     post
title:      Plume Gif 制作
subtitle:   根据WRF-LES模拟的nc文件生成不同排放流速率下、不同时间戳的柱浓度图像
date:       2024-01-10
author:     Peisipand
header-img: img/wallhaven-dg3opm.jpg
catalog: true
tags:
    - WRF
    - Matlab
---

# 一、初步测试

先写个简单的代码测试下单帧的柱浓度图像。

```
clc;clear
% 可能需要修改的量
pixel_res = 30; % 空间分辨率
emission_per_hour = 2000; % kg/h
M_gas = 16.04; % CO2:44  CH4:16.04
%% 第一步，得到缩放因子
n_total_air = 3.6055e+05; % 底面积是 1m2的时候，空气柱中的总分子数：in mol。该值的计算可以参照 IME气象资料的获取一文（https://peisipand.github.io/2024/01/10/Matlab_IME-meteorological-parameters/）
R = 8.314472; % in J/(mol·K)
file_path = 'H:\裴志鹏\LES-30m分辨率\500m_v4';
file_name_20min = [file_path,'\auxhist24_d01_0001-01-01_00_20_00'];
plume_20min = ncread(file_name_20min,'PLUME');  % 单位：单位总空气分子中 二氧化碳 的分子数。 换句话说，如果空气里全是二氧化碳，该值为1
PB = ncread(file_name_20min,'PB'); % Base-state pressure (Pa, 3d)
P = ncread(file_name_20min,'P'); % Perturbation pressure (Pa, 3d)
ptot = P + PB;
T = ncread(file_name_20min,'T');
t = (300+T) .* ( ptot ./ 100000) .^ (0.2854); % kappa = 0.2854.
PH = ncread(file_name_20min,'PH'); % Perturbation geopotential (m^2/s^2), 4d(sn * we * vertical * time)
PHB = ncread(file_name_20min,'PHB'); % Base-state geopotential (m^2/s^2), 4d(sn * we * vertical * time)
height = ( PH + PHB ) ./ 9.81; % compute height according geopotential
height = permute(height(1,1,:,1),[3 2 1]);
pressure = permute(ptot(1,1,:,1),[3 2 1]);
temp = permute(t(1,1,:,1),[3 2 1]);
% 根据20分钟处的二氧化碳质量，推算每h排放了多少二氧化碳，便于后面的单位变化。
R = 8.314472; % in J/(mol·K)
n_tot_air_each_level = pressure ./ (R .* temp) ; % 理想气体状态方程 PV=nRT  这里假设1m3
re_plume_20min = reshape(plume_20min,size(plume_20min,1) * size(plume_20min,2),50); %把三维的plume reshape成2维的
delta_height = diff(height);
n_tot_gas = reshape(re_plume_20min * (n_tot_air_each_level .* delta_height),size(plume_20min,1) , size(plume_20min,2)); % in mol，底面积为1m2，高度为整个空气柱
mass_tot_gas = sum(sum(pixel_res * pixel_res * n_tot_gas * M_gas / 1e3)); % 算上每个像素的面积，in kg
scale_factor = emission_per_hour / (mass_tot_gas * 3) ; %根据 mass_tot_ch4 / 0.33h 的排放速率对应的浓度场， 推算缩放因子

%% 第二步，读取时间戳=3h的PLUME，积分、缩放成柱浓度（3-D）
file_name = [file_path,'\auxhist24_d01_0001-01-01_03_00_00'];%输入文件路径
plume_3h = ncread(file_name,'PLUME');
PB = ncread(file_name,'PB'); % Base-state pressure (Pa, 3d)
P = ncread(file_name,'P'); % Perturbation pressure (Pa, 3d)
ptot = P + PB;
T = ncread(file_name,'T');
t = (300+T) .* ( ptot ./ 100000) .^ (0.2854); % kappa = 0.2854.
PH = ncread(file_name,'PH'); % Perturbation geopotential (m^2/s^2, 3d)
PHB = ncread(file_name,'PHB'); % Base-state geopotential (m^2/s^2, 3d)
height = ( PH + PHB ) ./ 9.81; % compute height according geopotential
height = permute(height(1,1,:),[3 2 1]);
pressure = permute(ptot(1,1,:),[3 2 1]);
temp = permute(t(100,1,:),[3 2 1]);
n_tot_air_each_level = pressure ./ (R .* temp) ; % 理想气体状态方程 PV=nRT  这里假设1m3
re_plume_3h = reshape(plume_3h,size(plume_3h,1) * size(plume_3h,2),50); % 把三维的plume reshape成2维的
delta_height = diff(height);
n_tot_gas = reshape(re_plume_3h * (n_tot_air_each_level .* delta_height),size(plume_3h,1) , size(plume_3h,2)); % in mol，底面积为1m2，高度为整个空气柱
n_tot_gas = n_tot_gas .* scale_factor; % 缩放后的摩尔数
column_3h = (n_tot_gas ./ n_total_air) .* 1e9 ; % in ppb
%% 第三步，绘制
imagesc(column_3h)
caxis([0 600])
%     m_colmap('jet','step',10)
mycolor = brewermap(5,'Reds');
mycolor = [[1 1 1];mycolor]; % [1 1 1] 表示纯白色，上一句中的 brewermap 未生成白色
xlabel('dx=30 m')
ylabel('dy=30 m')
title(['Q=2000 kg/h; V=2 m/s'])
colormap(mycolor)
h = colorbar;
set(h.Title,'string','XCH4 (ppb)');
%% 第四步，计算Q测试下
U10 = ncread(file_name,'U10');
U10 = U10(33,101);
V10 = ncread(file_name,'V10');
V10 = V10(33,101);
Wind_10 = sqrt(U10.^2 + V10.^2);
omega_a = 1.0374e4;
unenhanced_std = std(column_3h(:));
[Q,Q_std,mask] = func_IME(column_3h, omega_a, Wind_10,1);
```

![](\img\WRF_LES PLUME gif\plume_3h.jpg)

# 二、质量变化曲线

```
% mass_time_series.m
clc;clear
% 可能需要修改的量
pixel_res = 30; % 空间分辨率
emission_per_hour = 2000; % kg/h
M_gas = 16.04; % CO2:44  CH4:16.04
%% 第一步，得到缩放因子
n_total_air = 3.6055e+05; % 底面积是 1m2的时候，空气柱中的总分子数：in mol。该值的计算可以参照 IME气象资料的获取一文（https://peisipand.github.io/2024/01/10/Matlab_IME-meteorological-parameters/）
R = 8.314472; % in J/(mol·K)
file_path = 'H:\裴志鹏\LES-30m分辨率\500m_v2';
file_name_20min = [file_path,'\auxhist24_d01_0001-01-01_00_20_00'];
plume_20min = ncread(file_name_20min,'PLUME');  % 单位：单位总空气分子中 二氧化碳 的分子数。 换句话说，如果空气里全是二氧化碳，该值为1
PB = ncread(file_name_20min,'PB'); % Base-state pressure (Pa, 3d)
P = ncread(file_name_20min,'P'); % Perturbation pressure (Pa, 3d)
ptot = P + PB;
T = ncread(file_name_20min,'T');
t = (300+T) .* ( ptot ./ 100000) .^ (0.2854); % kappa = 0.2854.
PH = ncread(file_name_20min,'PH'); % Perturbation geopotential (m^2/s^2), 4d(sn * we * vertical * time)
PHB = ncread(file_name_20min,'PHB'); % Base-state geopotential (m^2/s^2), 4d(sn * we * vertical * time)
height = ( PH + PHB ) ./ 9.81; % compute height according geopotential
height = permute(height(1,1,:,1),[3 2 1]);
pressure = permute(ptot(1,1,:,1),[3 2 1]);
temp = permute(t(1,1,:,1),[3 2 1]);
% 根据20分钟处的二氧化碳质量，推算每h排放了多少二氧化碳，便于后面的单位变化。
R = 8.314472; % in J/(mol·K)
n_tot_air_each_level = pressure ./ (R .* temp) ; % 理想气体状态方程 PV=nRT  这里假设1m3
re_plume_20min = reshape(plume_20min,size(plume_20min,1) * size(plume_20min,2),50); %把三维的plume reshape成2维的
delta_height = diff(height);
% layer_height = ([0;delta_height] + [delta_height;0]) ./ 2;
n_tot_gas = reshape(re_plume_20min * (n_tot_air_each_level .* delta_height),size(plume_20min,1) , size(plume_20min,2)); % in mol，底面积为1m2，高度为整个空气柱
% sum(sum(n_tot_ch4))
mass_tot_gas = sum(sum(pixel_res * pixel_res * n_tot_gas * M_gas / 1e3)); % 算上每个像素的面积，in kg
scale_factor = emission_per_hour / (mass_tot_gas * 3) ; %根据 mass_tot_ch4 / 0.33h 的排放速率对应的浓度场， 推算缩放因子

%% 第二步，读取各个时间戳的PLUME，积分、缩放成柱浓度（3-D）
file_list = dir([file_path,'\','auxhist24*']);
n_plumes = length(file_list);
column = zeros(201,201,n_plumes);
mass = zeros(n_plumes,1);
for j = 1:n_plumes
    file_name = [file_path,'\',file_list(j).name];%输入文件路径
    plume_temp = ncread(file_name,'PLUME');
    PB = ncread(file_name,'PB'); % Base-state pressure (Pa, 3d)
    P = ncread(file_name,'P'); % Perturbation pressure (Pa, 3d)
    ptot = P + PB;
    T = ncread(file_name,'T');
    t = (300+T) .* ( ptot ./ 100000) .^ (0.2854); % kappa = 0.2854.
    PH = ncread(file_name,'PH'); % Perturbation geopotential (m^2/s^2, 3d)
    PHB = ncread(file_name,'PHB'); % Base-state geopotential (m^2/s^2, 3d)
    height = ( PH + PHB ) ./ 9.81; % compute height according geopotential
    height = permute(height(1,1,:),[3 2 1]);
    pressure = permute(ptot(1,1,:),[3 2 1]);
    temp = permute(t(100,1,:),[3 2 1]);
    n_tot_air_each_level = pressure ./ (R .* temp) ; % 理想气体状态方程 PV=nRT  这里假设1m3
    re_plume_temp = reshape(plume_temp,size(plume_temp,1) * size(plume_temp,2),50); % 把三维的plume reshape成2维的
    delta_height = diff(height);
    n_total_air = 3.6055e+05; % 底面积是 1m2的时候，空气柱中的总分子数：in mol(该值的计算可以参照 IME气象资料的获取一文)
    n_tot_gas = reshape(re_plume_temp * (n_tot_air_each_level .* delta_height),size(plume_temp,1) , size(plume_temp,2)); % in mol，底面积为1m2，高度为整个空气柱
    n_tot_gas = n_tot_gas .* scale_factor; % 缩放后的摩尔数
    column_temp = (n_tot_gas ./ n_total_air) .* 1e9 ; % in ppb
    mask = column_temp > 30; % 卫星探测精度为 20 ppm
    n_tot_gas = n_tot_gas .* mask;
    mass_tot_gas = sum(sum(pixel_res * pixel_res * n_tot_gas * M_gas / 1e3)); % 算上每个像素的面积，in kg
    mass(j,1) = mass_tot_gas;
end
plot(mass)
ylabel('kg')
xlabel('min')
```

![](\img\WRF_LES PLUME gif\mass_time_series.jpg)

# 三、Gif制作

```
%% generate_gif.m
%% 制作gif
clc;clear
% 可能需要修改的量
pixel_res = 30; % 空间分辨率
emission_per_hour = 2000; % kg/h
M_gas = 16.04; % CO2:44  CH4:16.04
%% 第一步，得到缩放因子
n_total_air = 3.6055e+05; % 底面积是 1m2的时候，空气柱中的总分子数：in mol。该值的计算可以参照 IME气象资料的获取一文（https://peisipand.github.io/2024/01/10/Matlab_IME-meteorological-parameters/）
R = 8.314472; % in J/(mol·K)
file_path = 'H:\裴志鹏\LES-30m分辨率\500m_v2'; % 边界层高度500 m；风速2 m/s
file_name_20min = [file_path,'\auxhist24_d01_0001-01-01_00_20_00'];
plume_20min = ncread(file_name_20min,'PLUME');  % 单位：单位总空气分子中 二氧化碳 的分子数。 换句话说，如果空气里全是二氧化碳，该值为1
PB = ncread(file_name_20min,'PB'); % Base-state pressure (Pa, 3d)
P = ncread(file_name_20min,'P'); % Perturbation pressure (Pa, 3d)
ptot = P + PB;
T = ncread(file_name_20min,'T');
t = (300+T) .* ( ptot ./ 100000) .^ (0.2854); % kappa = 0.2854.
PH = ncread(file_name_20min,'PH'); % Perturbation geopotential (m^2/s^2), 4d(sn * we * vertical * time)
PHB = ncread(file_name_20min,'PHB'); % Base-state geopotential (m^2/s^2), 4d(sn * we * vertical * time)
height = ( PH + PHB ) ./ 9.81; % compute height according geopotential
height = permute(height(1,1,:,1),[3 2 1]);
pressure = permute(ptot(1,1,:,1),[3 2 1]);
temp = permute(t(1,1,:,1),[3 2 1]);
% 根据20分钟处的二氧化碳质量，推算每h排放了多少二氧化碳，便于后面的单位变化。
R = 8.314472; % in J/(mol·K)
n_tot_air_each_level = pressure ./ (R .* temp) ; % 理想气体状态方程 PV=nRT  这里假设1m3
re_plume_20min = reshape(plume_20min,size(plume_20min,1) * size(plume_20min,2),50); %把三维的plume reshape成2维的
delta_height = diff(height);
% layer_height = ([0;delta_height] + [delta_height;0]) ./ 2;
n_tot_gas = reshape(re_plume_20min * (n_tot_air_each_level .* delta_height),size(plume_20min,1) , size(plume_20min,2)); % in mol，底面积为1m2，高度为整个空气柱
% sum(sum(n_tot_ch4))
mass_tot_gas = sum(sum(pixel_res * pixel_res * n_tot_gas * M_gas / 1e3)); % 算上每个像素的面积，in kg
scale_factor = emission_per_hour / (mass_tot_gas * 3) ; %根据 mass_tot_ch4 / 0.33h 的排放速率对应的浓度场， 推算缩放因子

%% 第二步，读取各个时间戳的PLUME，积分、缩放成柱浓度（3-D）
file_list = dir([file_path,'\','auxhist24*']);
n_plumes = length(file_list);
column = zeros(201,201,n_plumes);
for j = 1:n_plumes
    file_name = [file_path,'\',file_list(j).name];%输入文件路径
    plume_temp = ncread(file_name,'PLUME');
    PB = ncread(file_name,'PB'); % Base-state pressure (Pa, 3d)
    P = ncread(file_name,'P'); % Perturbation pressure (Pa, 3d)
    ptot = P + PB;
    T = ncread(file_name,'T');
    t = (300+T) .* ( ptot ./ 100000) .^ (0.2854); % kappa = 0.2854.
    PH = ncread(file_name,'PH'); % Perturbation geopotential (m^2/s^2, 3d)
    PHB = ncread(file_name,'PHB'); % Base-state geopotential (m^2/s^2, 3d)
    height = ( PH + PHB ) ./ 9.81; % compute height according geopotential
    height = permute(height(1,1,:),[3 2 1]);
    pressure = permute(ptot(1,1,:),[3 2 1]);
    temp = permute(t(100,1,:),[3 2 1]);
    n_tot_air_each_level = pressure ./ (R .* temp) ; % 理想气体状态方程 PV=nRT  这里假设1m3
    re_plume_temp = reshape(plume_temp,size(plume_temp,1) * size(plume_temp,2),50); % 把三维的plume reshape成2维的
    delta_height = diff(height);
    n_total_air = 3.6055e+05; % 底面积是 1m2的时候，空气柱中的总分子数：in mol(该值的计算可以参照 IME气象资料的获取一文)
    n_tot_gas = reshape(re_plume_temp * (n_tot_air_each_level .* delta_height),size(plume_temp,1) , size(plume_temp,2)); % in mol，底面积为1m2，高度为整个空气柱
    n_tot_gas = n_tot_gas .* scale_factor; % 缩放后的摩尔数
    column(:,:,j) = (n_tot_gas ./ n_total_air) .* 1e9 ; % in ppb
end

%% 第三步，绘制Gif
pic_num = 1;
for m = 1:n_plumes
    imagesc(column(:,:,m));
    caxis([0 600])
    mycolor = brewermap(5,'Reds');
    mycolor = [[1 1 1];mycolor]; % [1 1 1] 表示纯白色，上一句中的 brewermap 未生成白色
    xlabel('dx')
    ylabel('dy')
    date = file_list(m).name;
    date = [date(23:24),'\',date(25:27),'\',date(28:30)];
    title(['Q=2000 kg/h; V=2 m/s; Time:',date])
    %     zlim([0,0.008])
    %         caxis([0 800]);
    colormap(mycolor)
    h = colorbar;
    h.Ticks = 0:100:600;
    set(get(h,'Title'),'string','ppb');
    pause(0.004); % matlab展示的停顿时间
    drawnow;
    F=getframe(gcf);
    I=frame2im(F);
    [I,map]=rgb2ind(I,256);
    outfile = ['export_tif\','500m_v2_Q2000.gif'];
    dt = 0.004; % gif的停顿时间
    if pic_num == 1
        imwrite(I,map,outfile,'gif','Loopcount',1,'DelayTime',dt); % 'Loopcount',Inf 表示无限循环
    else
        imwrite(I,map,outfile,'gif','WriteMode','append','DelayTime',dt);
    end
    pic_num = pic_num + 1;
end
close
```

效果如图

![](\img\WRF_LES PLUME gif\500m_v2_Q2000.gif)

# 引用

如果你使用了这些代码，请对以下论文施加引用：

> Pei Z, Han G, Mao H, et al. Improving quantification of methane point source emissions from imaging spectroscopy[J]. Remote Sensing of Environment, 2023, 295: 113652.