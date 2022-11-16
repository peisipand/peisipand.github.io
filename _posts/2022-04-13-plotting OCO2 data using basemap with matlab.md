---
layout:     post
title:      利用Matlab在卫星底图上绘制OCO数据
subtitle:   Matlab提供了接口可以在线绘制各种底图，这里以绘制OCO-2卫星的XCO2为例进行展示。
date:       2022-04-13
author:     Peisipand
header-img: img/art-Anaconda-TensorFlow.jpg
catalog: true
tags:
    - Matlab
    - OCO
---






---

# 一、OCO-2数据读取
**[这里](https://disc.gsfc.nasa.gov/datasets/OCO2_L2_Diagnostic_10r/summary?keywords=OCO2)可以下载OCO-2的数据,批量下载的操作可以参考[这里](https://blog.csdn.net/peisipand/article/details/108224156?spm=1001.2014.3001.5501)。**

OCO-2的数据通常以nc和h5的格式存储，h5较于nc存储了更多的信息。
## 1.1以下展示了nc的读取，并用m_map工具箱进行绘制。
```bash
clc;
clear;
filename = 'F:\研究生\深圳碳中和会议\experiment\data\oco2_LtCO2_180225_B10206Ar_200730003152s.nc4';
lat = ncread(filename,'latitude');
lon = ncread(filename,'longitude');
xco2 = ncread(filename,'xco2');
windspeed_u_met = ncread(filename,'Meteorology/windspeed_u_met');
windspeed_v_met = ncread(filename,'Meteorology/windspeed_v_met');

lon_limit = [118 122];
lat_limit = [38 41];
id = (lat > lat_limit(1)) & (lat < lat_limit(2)) & (lon > lon_limit(1)) & (lon < lon_limit(2));%筛选出研究区内的数据
all = [lat(id,:) lon(id,:) xco2(id,:) windspeed_u_met(id,:) windspeed_v_met(id,:)];

figure(1)
m_proj('mercator', 'longitudes', lon_limit, 'latitudes', lat_limit)
m_grid('box','on', 'linestyle', 'none', 'tickdir', 'out', 'linewidth', 3, 'FontName','Times New Roman','fontsize',15);
m_mapshow('E:\MeteoInfo\map\country.shp')
hold on;
m_scatter(all(:,2),all(:,1),30,all(:,3) ,'filled', 'MarkerFaceColor', 'flat', 'MarkerEdgeColor', 'w','linewi',1) ;% lon, lat, 点的大小， 点的颜色映射， 实心
colormap(m_colmap('jet','step',10))
colorbar('FontName','Times New Roman','fontsize',15)
caxis([407 414])
```
绘制如图所示
![picture1](/img/OCO-basemap/plot_nc.jpg)

## 1.2下面是读取h5。
```bash
clc;
clear;
filename = 'F:\研究生\主动反演\OCO\利雅得\oco2_L2DiaGL_02624a_141229_B10004r_200111134405.h5';  %oco2_L2DiaGL_25017a_190316_B10004r_200501022851
% h5disp(filename);
% attribute = h5readatt('文件名.hdf', '数据集名', '属性名');
% data = h5read('文件名.hdf', '数据集名');
xco2 = double(h5read(filename,'/RetrievalResults/xco2'));
co2_profile = double(h5read(filename,'/RetrievalResults/co2_profile'));
lat = double(h5read(filename,'/RetrievalGeometry/retrieval_latitude'));
lon = double(h5read(filename,'/RetrievalGeometry/retrieval_longitude'));
```


# 二、添加卫星底图


```bash
gx = geoaxes
geoscatter(all(:,1),all(:,2),2,all(:,3))
caxis([407 414])
geobasemap streets
% geobasemap satelite
geolimits('manual')
geolimits(lat_limit,lon_limit)
```
![picture1](/img/OCO-basemap/plot_satelite.jpg)
