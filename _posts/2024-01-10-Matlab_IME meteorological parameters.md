---
layout:     post
title:      IME算法中气象资料的获取
subtitle:   IME算法中需要用到干空气柱积分质量和有效风速，这篇主要介绍如何获得这些数据。
date:       2024-01-10
author:     Peisipand
header-img: img/wallhaven-dg3opm.jpg
catalog: true
tags:
    - WRF
    - Matlab
---



IME算法中，需要用到的气象参数有：

> 1. 干空气柱积分质量 vertical_mass_of_atmosphere 
> 2. 有效风速 Effective wind speed

# 一、干空气柱积分质量

ERA5官网中：single level 提供了空气柱积分质量，尽管不是干空气的，但差异不大；pressure_level 提供了不同高度处的温湿压等参数，我们可以手动来对干空气的质量进行积分。



```
% 干空气柱积分质量下载地址：https://cds.climate.copernicus.eu/cdsapp#!/dataset/reanalysis-era5-single-levels?tab=form
%%
% 计算干空气的柱浓度 column concentration of dry air in kg/m2
% The molar mass of dry air is 28.9647 g/mol.
clc;
clear;
% single level
filename_EC_single_level = 'F:\LES\EC_single_level.nc'; % 2021-10-14 03:00
vertical_mass_of_atmosphere = ncread(filename_EC_single_level,'p53.162');  % in kg/m2   Vertical integral of mass of atmosphere
vertical_mass_of_atmosphere = vertical_mass_of_atmosphere(1,1);
sp = ncread(filename_EC_single_level,'sp'); % in Pa
sp = sp(1,1);
st = ncread(filename_EC_single_level,'t2m'); % in K
st = st(1,1);
geopotential_surface = ncread(filename_EC_single_level,'z');
geopotential_surface = geopotential_surface(1,1) ./ 9.81;
% pressure level
filename_EC_pressure_level = 'EC_pressure_level.nc';
q = ncread(filename_EC_pressure_level,'q');
q = permute(q(1,1,:),[3 2 1]);
t = ncread(filename_EC_pressure_level,'t');
t = permute(t(1,1,:),[3 2 1]);
T = [t ; st]; % in K
level = ncread(filename_EC_pressure_level,'level');
P = [double(level) * 100;sp]; % in Pa
geopotential = ncread(filename_EC_pressure_level,'z');
geopotential = permute(geopotential(1,1,:),[3 2 1]);
height = geopotential./ 9.81; % in meter
delta_height = -diff([height;0]);
layer_height = ([0;delta_height] + [delta_height;0]) ./ 2;
R = 8.314472; % in J/(mol·K)
% Avogadro = 6.02214076 * 1e23;
n = P ./ (R .* T) ; % 理想气体状态方程 PV=nRT  这里假设1m3
n_total_per_level = n .* layer_height; % in mol
n_total_air = sum(n_total_per_level); % 3.6055e+05 mol，之后会用到该值
vertical_mass_of_atmosphere_by_pzp = sum(M_dry * n_total_per_level) ./ 1e3; % in kg
```

# 二、有效风速 Ueff

根据Varon的论文的介绍，Ueff与U10之间的关系是通过拟合得到的，具体如何拟合参见上一篇博客。这里只介绍如何获取10米风速。

## 2.1 从GEOS-FP获取

```
% get_wind_speed_from_GEOS.m
function [wind_speed,u10,v10] = get_wind_speed_from_GEOS(year,month,day,hour,point_lat,point_lon)
%% 输入的 year,month,day,hour 需要是 字符类型，不过是double类型，下面的代码会进行修改
%% 注意：GEOS-FP的下载地址变了
if strcmp(class(year), 'double')
    year = num2str(year);
    month = num2str(month);
    if length(month) == 1
        month = ['0',month];
    end
    day = num2str(day);
    if length(day) == 1
        day = ['0',day];
    end
    hour = num2str(hour);
    if length(hour) == 1
        hour = ['0',hour];
    end
end
filename_wind =['wind_data_GEOS\GEOS_',year,'_',month,'_',day,'_',hour,'.nc4']; % 下载：https://portal.nccs.nasa.gov/cgi-lats4d/webform.cgi?&i=GEOS-5/fp/0.25_deg/assim/tavg1_2d_slv_Nx
if ~exist(filename_wind,'file')
    disp('正在下载GEOS-5风速数据...');
    URL = ['https://portal.nccs.nasa.gov/datashare/gmao/geos-fp/das/Y',year,'/M',month,'/D',day,'/GEOS.fp.asm.tavg1_2d_slv_Nx.',year,month,day,'_',hour,'30.V01.nc4'];
    disp(URL);
    system(['wget ',URL,' -O',filename_wind])
    %     urlwrite(URL, filename_wind);
    disp('下载完成');
end
% info = ncinfo(filename_wind);
lon_coarse  = ncread(filename_wind,'lon');
lat_coarse  = ncread(filename_wind,'lat');
u10 = ncread(filename_wind,'U10M');
u10 = flipud(u10');
v10 = ncread(filename_wind,'V10M');
v10 = flipud(v10');

[index_wind_lat,index_wind_lon] = find_index_based_coor(lat_coarse,lon_coarse,point_lat,point_lon);
wind_speed = sqrt( u10(index_wind_lat,index_wind_lon).^2 + v10(index_wind_lat,index_wind_lon).^2);
end
```

## 2.2 从EC获取

```
% get_wind_speed_from_EC.m
function [wind_speed,mass_of_atmos,u10,v10] = get_wind_speed_from_EC(year,month,day,hour,point_lat,point_lon)
if strcmp(class(year), 'char')
    year = str2double(year);
    month = str2double(month);
    day = str2double(day);
    hour = str2double(hour);
end
filename_wind =['wind_data_EC\EC_200101_230906.nc']; % 下载：https://portal.nccs.nasa.gov/cgi-lats4d/webform.cgi?&i=GEOS-5/fp/0.25_deg/assim/tavg1_2d_slv_Nx
% 下载
% info = ncinfo(filename_wind);
mass_of_atmos = ncread(filename_wind,'p53.162');
passed_hours = ncread(filename_wind,'time');
passed_hours = double(passed_hours);
date1 = datetime(1900, 1, 1, 0, 0, 0); % 1900年1月1日0时
date2 = datetime(year, month, day, hour, 0, 0); % 某年某月某日某时
% 计算两个日期之间的小时差
hoursDifference = hours(date2 - date1);
% 计算每个元素与要查找的数字的差值的绝对值
difference = abs(passed_hours - hoursDifference);
% 找到最小差值的索引
index_time = find(difference == min(difference));
mass_of_atmos = mass_of_atmos(:,:,1,index_time)';
lon_coarse  = ncread(filename_wind,'longitude');
lat_coarse  = ncread(filename_wind,'latitude');
u10 = ncread(filename_wind,'u10');
u10 = u10(:,:,1,index_time)';
v10 = ncread(filename_wind,'v10');
v10 = v10(:,:,1,index_time)';
%% 用 find_index_based_coor 确定行列号的前提是：通过转置和颠倒实现 imagesc(u10)和 Panoply绘制的结果一样
[index_wind_lat,index_wind_lon] = find_index_based_coor(lat_coarse,lon_coarse,point_lat,point_lon);
wind_speed = sqrt( u10(index_wind_lat,index_wind_lon).^2 + v10(index_wind_lat,index_wind_lon).^2);
mass_of_atmos = mass_of_atmos(index_wind_lat,index_wind_lon);
end
```

## 2.3 获取某经纬度处的风速

以上两个函数均用到了 find_index_based_coor 函数，该函数根据输入的经纬度返回行列号。

```
function [index_lat,index_lon] = find_index_based_coor(lat,lon,point_lat,point_lon,num_lat,num_lon)
if nargin <= 4 % 该情况表示提供的lat/lon不是一个矩阵，而是只有起始、终止值。
    num_lat = length(lat);
    num_lon = length(lon);
end
min_lat = min(lat);
max_lat = max(lat);
min_lon = min(lon);
max_lon = max(lon);
step_lat = (max_lat - min_lat) / (num_lat - 1);
step_lon = (max_lon - min_lon) / (num_lon - 1);
index_lat = floor((max_lat - point_lat) / step_lat) + 1 ;
index_lon = floor((point_lon - min_lon) / step_lon) + 1;
end
```



## 2.4 比较以上两个数据集

```
clc;clear;
year = '2023';
month = '02';
day = '04';
hour = '03';

for i = 1:16
    for j = 1:16
        [wind_speed,u10,v10] = get_wind_speed_from_GEOS(year,month,day,hour,36 + i * 0.25 ,110 + j * 0.25);
        ws_GEOS(i,j) = wind_speed;
        [wind_speed,mass_of_atmos,u10,v10] = get_wind_speed_from_EC(year,month,day,hour,36 + i * 0.25 ,110 + j * 0.25);
        ws_EC(i,j) = wind_speed;
    end
end

ws_GEOS = reshape(ws_GEOS,16*16,1);
ws_EC = reshape(ws_EC,16*16,1);
scatter(ws_GEOS,ws_EC)
xlabel('GEOS')
ylabel('EC')
```

![picture 1](\img\Matlab_IME meteorological parameters\scatter_GEOS_EC.jpg)

这段代码需要运行挺久，耐心等待即可。可以看出：在该位置，GEOS的风速较于EC偏高。

