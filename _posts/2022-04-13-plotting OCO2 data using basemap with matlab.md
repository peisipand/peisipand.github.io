---
layout:     post
title:      基于MATLAB的地理数据空间可视化
subtitle:   本文提供了三种方式来可视化空间数据。
date:       2022-04-13
author:     Peisipand
header-img: img/art-Anaconda-TensorFlow.jpg
catalog: true
tags:
    - Matlab
    - OCO
    - PRISMA
---

---

# 一、geoshow on on regular axes

**绘制的主要流程：**
1. 创建地图坐标区域(通过axesm、worldmap、usamap)
2. 导入数据(通过load或者shaperead)
3. 通过geoshow、plotm、scatterm、patchm绘制图像(大部分普通坐标区域axes中出现的函数加个m就变成了用于地图坐标区域的函数)

```
clc;clear
load korea5c % 数据导入
figure('Name', 'Korea地形图', 'Units','centimeters','Position',[10 10 9 9]); %输出宽度为9厘米的图
ax= worldmap(korea5c,korea5cR) % 缺少这句话直接运行会导致地图上不显示经纬度
geoshow(korea5c,korea5cR,'DisplayType','texturemap')
demcmap(korea5c)
```
![picture1](/img/OCO-basemap/korea.jpg)


**数据导入部分：**

其中可以通过load导入的MATLAB自带数据有：
1. coastlines - 世界海岸线经纬度矢量
2. conus - 用于连接的美国(conus)、五大湖、州际边界的经纬度阵列
3. geoid60c - 全球大地水准面高度网格（以米为单位）/度
4. greatlakes - 显示结构阵列中的北美五大湖
5. korea5c - 朝鲜半岛的地形和水深测量
6. koreaEQdata - 地震位置和震级
7. layermtx - 用于教学的地理定位地形网格
8. mapmtx - 用于教学的地理定位地形网格
9. moonalb20c - 克莱门汀全球月球反照率图
10. moontopo60c - 月球的克莱门汀激光雷达地形
11. oceanlo - 显示结构数组中的海洋遮罩多边形
12. russia - 网格化土地、水域、边界、外部区域
13. seatempm -全球多通道海面温度网格
14. stars - 4500+颗恒星的天体坐标和星等
15. usamtx - 美国各州的数据网格，每度五个单元格
16. usgslulegend - USGS 土地利用类别列表

可以通过shaperead导入的MATLAB自带数据有：

1. landareas.shp - 全球陆地区域多边形
2. tsunamis.shp - 全球1950-2006 年中到大型海啸的百分比
3. usastatehi.shp - 高分辨率多边形美国各州形状
4. usastatelo.shp - 低多边形美国各州形状
5. worldcities.shp - 全球318个城市或人口稠密位置坐标
6. worldlakes.shp - 世界上 37 个最大的多边形湖泊和内陆海域
7. worldrivers.shp - 世界主要河流的线条形状
8. boston_placenames.shp - 美国马萨诸塞州波士顿地名
9. boston_roads.shp - 美国马萨诸塞州波士顿道路
10. concord_hydro_area.shp - 美国马萨诸塞州康科德水域
11. concord_hydro_line.shp - 美国马萨诸塞州康科德水路
12. concord_roads.shp - 美国马萨诸塞州康科德道路

**绘制陆地区域、湖泊、河流、城市的世界地图**

```
clc;clear
% 创建世界地图坐标区域
ax=worldmap('World'); % worldmap()里面支持的参数，可以通过运行一下函数worldmap()就会有提示
setm(ax,'Origin',[0 180 0]) % 中间的经纬度调整为180
% 绘制陆地
land=shaperead('landareas.shp','UseGeoCoords',true);
geoshow(ax,land,'FaceColor',[0.5 0.7 0.5])
% 绘制湖泊
lakes=shaperead('worldlakes.shp','UseGeoCoords',true);
geoshow(ax,lakes,'FaceColor','blue')
% 绘制河流
rivers=shaperead('worldrivers.shp','UseGeoCoords',true);
geoshow(ax,rivers, 'Color', 'blue')
% 绘制城市
cities=shaperead('worldcities.shp','UseGeoCoords',true);
geoshow(ax,cities,'Marker','.','Color','red')
```
![picture1](/img/OCO-basemap/world.jpg)


同时worldmap函数支持直接输入经纬度范围，例如：

```cpp
latlim=[3 54];
lonlim=[73 135];
worldmap(latlim,lonlim)
```
![picture1](/img/OCO-basemap/world_lim.jpg)

```
clc;clear
% 创建世界地图坐标区域并将区域设置为南极洲
worldmap('antarctica')
% 从陆地区域数据文件中获取南极洲大陆数据并绘图
antarctica = shaperead('landareas.shp', 'UseGeoCoords', true,...
  'Selector',{@(name) strcmp(name,'Antarctica'), 'Name'});
patchm(antarctica.Lat, antarctica.Lon, [0.5 1 0.5]) %把polygon边界的经纬度连起来并用绿色填充
```
![picture1](/img/OCO-basemap/antarctica.jpg)

**绘制中国大陆陆地：**

```cpp
% 创建世界地图坐标区域并将区域设置为中国
worldmap('China')
% 获取陆地边界，蓝色描边
land = shaperead('landareas.shp', 'UseGeoCoords', true,...
  'Selector',{@(name) strcmp(name,'Africa and Eurasia'), 'Name'});
patchm(land.Lat, land.Lon, [0.5 0.7 0.5])
% 加个海岸线美化一下
load coastlines
plotm(coastlat,coastlon)
```
![picture1](/img/OCO-basemap/china.jpg)

**绘制水准面高度**

```
% 大地水准面高度数导入
load geoid60c.mat
% 创建某经纬度范围世界地图坐标区域
latlim=[-50 50];
lonlim=[160 -30];
ax=worldmap(latlim,lonlim);
% 设置颜色
C=[222,238,209;126,190,174;144,213,220;
    33,118,155;30,69,128;20,49,127]./255;
geoshow(ax,geoid60c,geoid60cR,'DisplayType','surface')
colormap(C) % 应用颜色
land=shaperead('landareas.shp','UseGeoCoords',true);
geoshow(ax,land,'FaceColor',[0.5 0.7 0.5])
```
![picture1](/img/OCO-basemap/geoid60c.jpg)
**绘制shp**
```
usamap("conus");
states=shaperead("usastatelo.shp",'UseGeoCoords',true);
% Alaska, Hawaii俩州离太远画不开，不要
for i=length(states):-1:1
    if states(i).Name=="Alaska"||states(i).Name=="Hawaii"
        states(i)=[];
    end
end
% 插值定义颜色
C=[222,238,209;126,190,174;144,213,220;
    33,118,155;30,69,128;20,49,127]./255;
C1(:,1)=interp1(0:5,C(:,1),linspace(0,5,numel(states)),'linear')';
C1(:,2)=interp1(0:5,C(:,2),linspace(0,5,numel(states)),'linear')';
C1(:,3)=interp1(0:5,C(:,3),linspace(0,5,numel(states)),'linear')';
faceColors=makesymbolspec('Polygon',{'INDEX',[1 numel(states)],'FaceColor',C1});
geoshow(states, 'DisplayType','polygon','SymbolSpec', faceColors)
```
![picture1](/img/OCO-basemap/usamap.jpg)

**axesm**

创建一个robinson样式，带框的地图坐标区域：

```cpp
axesm('MapProjection','robinson','Frame','on')
```

创建好的axesm可以通过setm修改样式：

```cpp
axesm('MapProjection','robinson','Frame','on')
setm(gca,'FLineWidth',3,'Grid','on')
```

创建一个ortho投影

```
clc;clear
% 某些视角下的地图坐标区,其他经线的纬度范围[-75 75],四条经线绘制完全
axesm('MapProjection','ortho','Frame','on',...
    'grid','on','Origin',[29,114,14],... % 29°N 114°E设置为地图中心，14度为视觉偏转角度
    'MLineLimit',[-75 75],...
    'MLineException',[-90,0,90,180])
% 导入海岸线数据
load coastlines
% 绘制海岸线
plotm(coastlat,coastlon)

```
![picture1](/img/OCO-basemap/ortho.jpg)

# 二、geoshow on GeographicAxes

优点：能显示高分辨的卫星（街道等等）底图

缺点：只能显示矢量数据，绘制不了栅格。

**下面以OCO-2数据为例进行绘制**

数据批量下载的操作可以参考[我之前的博客](https://blog.csdn.net/peisipand/article/details/108224156?spm=1001.2014.3001.5501)。

OCO-2的数据通常以nc和h5的格式存储，h5较于nc，存储了更多的信息。

```
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
%% 绘制
gx = geoaxes
% set(gca,'FontName','Times New Roman','FontSize',20) 
geoscatter(all(:,1),all(:,2),2,all(:,3))
caxis([407 414])
geobasemap streets
% geobasemap satelite
geolimits('manual')
geolimits(lat_limit,lon_limit)
```

![picture1](/img/OCO-basemap/plot_satelite.jpg)

## 读取h5的代码
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

# 三、m_map工具箱

优点：可以同时绘制栅格、矢量数据，并且比较好看

缺点：不能使用高分辨的卫星底图

## 3.1绘制Point矢量数据（以OCO-2为例）

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

## 3.2绘制栅格数据（以PRISMA为例）

```

clc;clear
filename = 'F:\研究生\高光谱甲烷识别\数据\山西5\PRS_L1_STD_OFFL_20210206031837_20210206031842_0001.he5';
lat = h5read(filename,'/HDFEOS/SWATHS/PRS_L1_HRC/Geolocation Fields/Latitude_SWIR');
lon = h5read(filename,'/HDFEOS/SWATHS/PRS_L1_HRC/Geolocation Fields/Longitude_SWIR');
ScaleFactor_Swir = h5readatt(filename,'/','ScaleFactor_Swir');
Offset_Swir = h5readatt(filename,'/','Offset_Swir');
swir = h5read(filename,'/HDFEOS/SWATHS/PRS_L1_HRC/Data Fields/SWIR_Cube');
swir = double(swir) / double(ScaleFactor_Swir) - double(Offset_Swir);
swir = permute(swir,[1 3 2]);
data = swir(:,:,90);
lat = double(lat); % 此处需要从single格式转成double格式
lon = double(lon);
m_proj('Mercator','lon',[min(lon(:)) max(lon(:))],'lat',[min(lat(:)) max(lat(:))]);%设置投影方式为：墨卡托，地图显示范围
m_pcolor(lon,lat,data);
hold on;
% m_ruler(1.03,[.15 .5],'ticklen',[.01]); % 竖向的比例尺 1.03表示比例尺的横坐标；0.15
% 0.5表示比例尺竖向的起始、终止的位置；ticklen表示标签线的长度
m_ruler([.7 1],.95,'ticklen',[.01],'fontsize',10); % 横向的比例尺
m_grid('fontsize',10,'box','on', 'tickdir', 'out', 'tickstyle','dd');
%     'xtick',2,...
%     'xticklabel',['112.7°E';'113.1°E'],...
%     'ytick',2,...
%     'yticklabel',['36.1°N';'36.4°N'],...
colormap(m_colmap('jet'));
caxis([0 10]);
xlabel('Longitude'); 
ylabel('Latitude'); 
title('CH4 plume'); 
set(gca,'fontname','times new roman')
```

![picture1](/img/OCO-basemap/PRISMA.jpg)
