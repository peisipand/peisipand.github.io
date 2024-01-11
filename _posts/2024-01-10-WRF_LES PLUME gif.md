---
layout:     post
title:      Plume Gif 制作
subtitle:   根据WRF-LES模拟的nc文件生成不同排放流速率下、不同时间戳的柱浓度图像
date:       2024-01-10
author:     Peisipand
header-img: img/art-Anaconda-TensorFlow.jpg
catalog: true
tags:
    - WRF
---

# 一、初步测试

先写个简单的代码测试下单帧的柱浓度图像。

```
% generate_gif.m
%% 初步测试
clc;clear
% 可能需要修改的量
pixel_res = 30; % 空间分辨率
emission_per_hour = 2000; % kg/h
M_gas = 16.04; % CO2:44  CH4:16.04
% 第一步，得到缩放因子
n_total = 3.7052e+05; % in mol
R = 8.314472; % in J/(mol·K)
file_path = 'H:\裴志鹏\LES-30m分辨率\v4';
file_list = dir([file_path,'\','auxhist24_d01_0001-01-01_00_20_00']);
file_name_20min = [file_list.folder,'\',file_list.name];
plume_1h = ncread(file_name_20min,'PLUME');  %单位：单位总空气分子中 二氧化碳的分子数。 换句话说，如果空气里全是二氧化碳，该值为1
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
n_tot_air = pressure ./ (R .* temp) ; % 理想气体状态方程 PV=nRT  这里假设1m3
re_plume_1h = reshape(plume_1h,size(plume_1h,1) * size(plume_1h,2),50); %把三维的plume reshape成2维的
delta_height = diff(height);
% layer_height = ([0;delta_height] + [delta_height;0]) ./ 2;
n_tot_gas = reshape(re_plume_1h * (n_tot_air .* delta_height),size(plume_1h,1) , size(plume_1h,2)); % in mol，底面积为1m2，高度为整个空气柱
% sum(sum(n_tot_ch4))
mass_tot_ch4 = sum(sum(pixel_res * pixel_res * n_tot_gas * M_gas / 1e3)); % 算上每个像素的面积，in kg
scale_factor = emission_per_hour / (mass_tot_ch4 * 3) ; %根据 mass_tot_ch4 / 0.33h 的排放速率对应的浓度场， 推算缩放因子
% 第二步
file_list = dir([file_path,'\','auxhist24_d01_0001-01-01_03_00_00']);
file_name_3h = [file_list.folder,'\',file_list.name];
plume_3h = ncread(file_name_3h,'PLUME');  %单位：单位总空气分子中 二氧化碳的分子数。 换句话说，如果空气里全是二氧化碳，该值为1
PB = ncread(file_name_3h,'PB'); % Base-state pressure (Pa, 3d)
P = ncread(file_name_3h,'P'); % Perturbation pressure (Pa, 3d)
ptot = P + PB;
T = ncread(file_name_3h,'T');
t = (300 + T) .* ( ptot ./ 100000) .^ (0.2854); % kappa = 0.2854.
PH = ncread(file_name_3h,'PH'); % Perturbation geopotential (m^2/s^2, 3d)
PHB = ncread(file_name_3h,'PHB'); % Base-state geopotential (m^2/s^2, 3d)
height = ( PH + PHB ) ./ 9.81; % compute height according geopotential
height = permute(height(1,1,:),[3 2 1]);
pressure = permute(ptot(1,1,:),[3 2 1]);
temp = permute(t(100,1,:),[3 2 1]);
% 根据3h处的二氧化碳质量，推算每h排放了多少二氧化碳，便于后面的单位变化。
n_tot_air = pressure ./ (R .* temp) ; % 理想气体状态方程 PV=nRT  这里假设1m3
re_plume_3h = reshape(plume_3h,size(plume_3h,1) * size(plume_3h,2),50); %把三维的plume reshape成2维的
delta_height = diff(height);
% layer_height = ([0;delta_height] + [delta_height;0]) ./ 2;
n_tot_gas = reshape(re_plume_3h * (n_tot_air .* delta_height),size(plume_3h,1) , size(plume_3h,2)); % in mol，底面积为1m2，高度为整个空气柱
n_tot_gas = n_tot_gas .* scale_factor; % 缩放后的总质量
column = (n_tot_gas ./ n_total) .* 1e9 ; % in ppb
imagesc(column)
caxis([0 600])
%     m_colmap('jet','step',10)
mycolor = uint8([255,255,255
    254,232,200
    253,212,158
    253,187,132
    252,141,89
    239,101,72
    215,48,31
    179,0,0
    127,0,0]);
xlabel('dx=30 m')
ylabel('dy=30 m')
title(['Input:Q=2000 kg/h.'])
colormap(mycolor)
h = colorbar;
```

![](F:\个人主页_博客\peisipand.github.io\img\WRF_LES PLUME gif\plume_3h.jpg)

# 二、质量变化曲线

```
% mass_time_series.m
clc;
clear;
pixel_res = 30; % 空间分辨率30米
% 第一步，得到缩放因子
file_list = dir(['H:\裴志鹏\LES-30m分辨率\v4\au','*00']);
for i = 1:size(file_list,1)
    file_name = [file_list(i).folder,'\',file_list(i).name];
    plume = ncread(file_name,'PLUME');  %单位：单位总空气分子中 二氧化碳的分子数。 换句话说，如果空气里全是二氧化碳，该值为1
    PB = ncread(file_name,'PB'); % Base-state pressure (Pa, 3d)
    P = ncread(file_name,'P'); % Perturbation pressure (Pa, 3d)
    ptot = P + PB;
    T = ncread(file_name,'T');
    t = (300+T) .* ( ptot ./ 100000) .^ (0.2854); % kappa = 0.2854.
    PH = ncread(file_name,'PH'); % Perturbation geopotential (m^2/s^2), 4d(sn * we * vertical * time)
    PHB = ncread(file_name,'PHB'); % Base-state geopotential (m^2/s^2), 4d(sn * we * vertical * time)
    height = ( PH + PHB ) ./ 9.81; % compute height according geopotential
    height = permute(height(1,1,:,1),[3 2 1]);
    pressure = permute(ptot(1,1,:,1),[3 2 1]);
    temp = permute(t(1,1,:,1),[3 2 1]);
    % 根据20分钟处的二氧化碳质量，推算每h排放了多少二氧化碳，便于后面的单位变化。
    M_CO2 = 44;
    n_total = 3.7052e+05; % in mol
    R = 8.314472; % in J/(mol·K)
    n_tot_air = pressure ./ (R .* temp) ; % 理想气体状态方程 PV=nRT  这里假设1m3
    re_plume_1h = reshape(plume,size(plume,1) * size(plume,2),50); %把三维的plume reshape成2维的
    delta_height = diff(height);
    % layer_height = ([0;delta_height] + [delta_height;0]) ./ 2;
    n_tot_co2 = reshape(re_plume_1h * (n_tot_air .* delta_height),size(plume,1) , size(plume,2)); % in mol，底面积为1m2，高度为整个空气柱
    % sum(sum(n_tot_ch4))
    mass_tot_ch4 = sum(sum(pixel_res * pixel_res * n_tot_co2 * M_CO2 / 1e3)); % 算上每个像素的面积，in kg
    mass(i) = mass_tot_ch4;
end
plot(mass)
mass(2) / 10
(mass(2) - mass(1)) / 10
```

# 三、Gif制作

```
% generate_gif.m
%% 制作gif
clc;clear
% 可能需要修改的量
pixel_res = 30; % 空间分辨率
emission_per_hour = 2000; % kg/h
M_gas = 16.04; % CO2:44  CH4:16.04
% 第一步，得到缩放因子
n_total = 3.7052e+05; % in mol
R = 8.314472; % in J/(mol·K)
file_path = 'H:\裴志鹏\LES-30m分辨率\v4';
file_list = dir([file_path,'\','*00']);
n = length(file_list);
for j = 1:n
    file_name = [file_path,'\',file_list(j).name];%输入文件路径
    plume = ncread(file_name,'PLUME');
    eval( ['plume', num2str(j), '=','plume;'] );
end
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
n_tot_air = pressure ./ (R .* temp) ; % 理想气体状态方程 PV=nRT  这里假设1m3
pic_num = 1;
for m = 1:n
    date = file_list(m).name;
    date = [date(23:24),'\',date(25:27),'\',date(28:30)];
    plume_h = eval(['plume',num2str(m)]);
    %imagesc(temp(:,:,10));
    % 算柱浓度
    re_plume = reshape(plume_h,size(plume_h,1) * size(plume_h,2),50);
    delta_height = diff(height);
    % layer_height = ([0;delta_height] + [delta_height;0]) ./ 2;
    n_tot_gas = reshape(re_plume * (n_tot_air .* delta_height),size(plume_h,1) , size(plume_h,2)); % in mol，底面积为1m2，高度为整个空气柱
    %     sum(sum(n_tot_ch4))
    mass_tot_ch4 = sum(sum(pixel_res * pixel_res * n_tot_gas * M_gas / 1e3)); % 算上每个像素的面积，in kg
    emission_per_hour = 1000; % in kg
    scale_factor = (mass_tot_ch4 / (emission_per_hour * 3)); % 除以3个小时的排放总量 得到缩放因子
    column = (n_tot_gas ./ n_total) .* 1e9 ./ scale_factor;
    imagesc(column);
    caxis([0 600])
    %     date = num2str(12 * m);
    %     imagesc(plume1(:,:,15,m));
    %     m_colmap('jet','step',10)
    mycolor = uint8([255,255,255
        254,232,200
        253,212,158
        253,187,132
        252,141,89
        239,101,72
        215,48,31
        179,0,0
        127,0,0]);
    
    xlabel('dx')
    ylabel('dy')
    title(['Input:Q=2000 kg/h. Time:',date])
    %     zlim([0,0.008])
    %         caxis([0 800]);
    colormap(mycolor)
    h = colorbar;
    set(get(h,'Title'),'string','ppb');
    pause(0.5); % 停顿时间
    
    drawnow;
    F=getframe(gcf);
    I=frame2im(F);
    [I,map]=rgb2ind(I,256);
    outfile = ['export_tif\','v4_30m_Q2000.gif'];
    dt = 0.4;
    if pic_num == 1
        imwrite(I,map,outfile,'gif','Loopcount',Inf,'DelayTime',dt);
    else
        imwrite(I,map,outfile,'gif','WriteMode','append','DelayTime',dt);
    end
    pic_num = pic_num + 1;
end
close
```

效果如图

![](F:\个人主页_博客\peisipand.github.io\img\WRF_LES PLUME gif\v4_30m_Q2000.gif)

# 引用

如果你使用了这些代码，请对以下论文施加引用：

> Pei Z, Han G, Mao H, et al. Improving quantification of methane point source emissions from imaging spectroscopy[J]. Remote Sensing of Environment, 2023, 295: 113652.