---
layout:     post
title:      基于MATLAB的地理数据空间可视化
subtitle:   本文提供了三种方式来可视化空间数据。
date:       2022-04-13
author:     Peisipand
header-img: img/wallhaven-dg3opm.jpg
catalog: true
tags:
    - Matlab
    - Plot
---


# 三种方法对比

<table>  
    <thead>    
        <tr>      
            <th></th>      
            <th>geoshow</th>
            <th>m_map工具箱</th>  
            <th>geoaxes</th>  
        </tr>  
    </thead>  
    <tbody>    
        <tr>      
            <td>Matlab自带函数</td>      
            <td>√</td>    
            <td>×</td>      
            <td>√</td> 
        </tr>  
        <tr>      
            <td>绘制栅格</td>      
            <td>√</td>    
            <td>√</td>      
            <td>×</td> 
        </tr>  
        <tr>      
            <td>绘制矢量</td>      
            <td>√</td>    
            <td>√</td>      
            <td>√</td> 
        </tr>  
        <tr>      
            <td>设置投影</td>      
            <td>√</td>    
            <td>√</td>      
            <td>×</td> 
        </tr>  
        <tr>      
            <td>设置底图</td>      
            <td>×</td>    
            <td>×</td>      
            <td>√</td> 
        </tr>  
        <tr>      
            <td>颜值</td>      
            <td>🌟🌟🌟</td>    
            <td>🌟🌟🌟🌟🌟</td>      
            <td>🌟🌟🌟</td> 
        </tr>  
    </tbody>
</table>


# 一、geoshow

## 1.1 geoshow 绘制 shp
1. 加载shp文件：
```
% Input *.shp file
cn_p='F:\个人主页_博客\peisipand.github.io\data\shp\china_basic_map_micaps4\bou2_4p.shp'; % plane data -- territory
cn_l='F:\个人主页_博客\peisipand.github.io\data\shp\china_basic_map_micaps4\bou2_4l.shp'; % line data -- border
bou2_4p=shaperead(cn_p, 'UseGeoCoords', true);
bou2_4l=shaperead(cn_l, 'UseGeoCoords', true);   
```
2. 定义地图的绘制范围：
```
mainland_latlim = [18 55];mainland_lonlim = [70 140]; % mainland lonlat range
```
3. 定义轴Ax(可以使用 ``axesm``, ``worldmap(latlim,lonlim)``, 或者 ``usamap()`` 函数创建)
```
% grid: 格网；frame: 外边框；
% MeridianLabel: 经度label；ParallelLabel: 纬度label；
% LabelRotation: Label跟着格网旋转一定角度
% MLineLocation: 经度间隔；PLineLocation: 纬度间隔
% MLabelParallel: 经度label 在上面或者下面显示（north or south）
% 字号和字体大小需要在此处设置，后续的set(gca)无法修改经纬度label的字体
ax = axesm('MapProjection','lambert',...
    'grid','on','frame','on',...
    'MeridianLabel','on','ParallelLabel','on',  ...
    'MapLatLimit',mainland_latlim,'MapLonLimit',mainland_lonlim,...
    'LabelRotation','off',... 
    'MLineLocation',10,'PLineLocation',10,...
    'MLabelParallel','south',...
    'fontname','Arial','fontsize',8); 
ax.Visible = 'off';  % 不加这句会导致：ax 有白色背景，不好看
```
4. 绘制 shp 数据，并设置颜色：
```
geoshow(bou2_4p,'Facecolor',[0.9,0.9,0.9]); % 灰色绘制polygon
geoshow([bou2_4l.Lat],[bou2_4l.Lon],'Color','k'); %黑色绘制边界线
```
5. 代码汇总
```
clc;clear
% Input *.shp file
cn_p='F:\个人主页_博客\peisipand.github.io\data\shp\china_basic_map_micaps4\bou2_4p.shp'; % plane data -- territory
cn_l='F:\个人主页_博客\peisipand.github.io\data\shp\china_basic_map_micaps4\bou2_4l.shp'; % line data -- border
bou2_4p=shaperead(cn_p, 'UseGeoCoords', true);
bou2_4l=shaperead(cn_l, 'UseGeoCoords', true);
mainland_latlim = [18 55];mainland_lonlim = [70 140]; % mainland lonlat range
figure;
set(gcf,'unit','centimeters','position',[25,15,14,11.4]); % 图形窗口在电脑屏幕上的位置和尺寸[左 下 宽 高]
ax = axesm('MapProjection','lambert',...
    'grid','on','frame','on',...
    'MeridianLabel','on','ParallelLabel','on',  ...
    'MapLatLimit',mainland_latlim,'MapLonLimit',mainland_lonlim,...
    'LabelRotation','off',... 
    'MLineLocation',10,'PLineLocation',10,...
    'MLabelParallel','south',...
    'fontname','Arial','fontsize',8); 
% grid: 格网；frame: 外边框；
% MeridianLabel: 经度label；ParallelLabel: 纬度label；
% LabelRotation: Label跟着格网旋转一定角度
% MLineLocation: 经度间隔；PLineLocation: 纬度间隔
% MLabelParallel: 经度label 在上面或者下面显示（north or south）
getm(gca)% 可以查看axesm的所有参数
% setm(gca,'fontname','Arial','fontsize',8) %可以改变参数
ax.Visible = 'off'; %不加这句话 ax 会有白色背景，不好看
geoshow(bou2_4p,'Facecolor',[0.9,0.9,0.9]); % 灰色绘制polygon
geoshow([bou2_4l.Lat],[bou2_4l.Lon],'Color','k'); %黑色绘制边界线
print(gcf, 'china_shp.png', '-dpng', '-r300'); % 保存为PNG格式，分辨率为 300 dpi
```
![Alt text](/img/Matlab_geoshow/china_shp.png)

## 1.2 geoshow 绘制 georaster
有时候，我们有全球的 tif 文件，但是只想绘制中国区域的 tif，一种做法是提前在 envi 中裁剪好再来绘制。还可以利用代码来白化，之所以用[白化](http://i-lightning.cn/2020/05/matlab_mask/)这个词，是从一个大佬那里学的。
1. 对于 tif 文件，我们可以利用``readgeoraster``函数，读取出 data 和其对应的 dataR 。这里以绘制matlab自带的数据为例：
```
clear, clc
load korea5c
figure;
set(gcf,'unit','centimeters','position',[25,15,14,11.4]); % 图形窗口在电脑屏幕上的位置和尺寸[左 下 宽 高]
ax = axesm('MapProjection','ortho','Frame','on',...
    'grid','on','Origin',[29,114,14],... % 29°N 114°E设置为地图中心，14度为视觉偏转角度
    'MLineLimit',[-75 75],...
    'MLineException',[-90,0,90,180],...
    'fontname','Arial','fontsize',8)
getm(gca)
ax.Visible = 'off'
% 导入海岸线数据
load coastlines
% 绘制海岸线
plotm(coastlat,coastlon)
geoshow(korea5c,korea5cR,'DisplayType','texturemap')
demcmap(korea5c) % 根据korea5c生成 colormap N*3 矩阵
cb = colorbar('southoutside','fontname','Arial','fontsize',8);
cb.Label.String = 'Label';
print(gcf, 'korea5c.png', '-dpng', '-r300'); % 保存为PNG格式，分辨率为 300 dpi
```
![Alt text](/img/Matlab_geoshow/korea5c.png)

2. 对于 nc 文件，我们可以利用``ncread``函数，读取出 data 和其对应的 lon、 lat 。这里以绘制 [ODIAC（1 degree nc）](https://db.cger.nies.go.jp/nies_data/10.17595/20170411.001/odiac2022/1deg_ncdf/odiac2022_1x1d_2021.nc) 的 数据为例：
```
clear, clc
file_name = 'E:\谷歌下载\odiac2022_1x1d_2021.nc';
data = ncread(file_name,'land');% in gC/m2/d
data = double(data(:,:,1))'; % 取 第1个月
% data(data == 0) = nan;
lon = ncread(file_name,'lon');
lat = ncread(file_name,'lat');
[LON,LAT] = meshgrid(linspace(lon(1),lon(end),size(lon,1)), linspace(lat(1),lat(end),size(lat,1))); 
figure;
set(gcf,'unit','centimeters','position',[25,15,14,11.4]); % 图形窗口在电脑屏幕上的位置和尺寸[左 下 宽 高]
ax = axesm('MapProjection','eckert4','fontname','Arial','fontsize',8);
getm(gca)
ax.Visible = 'off';
% 导入海岸线数据
load coastlines
% 绘制海岸线
plotm(coastlat,coastlon);
geoshow(LAT,LON,data,'DisplayType','surface')%当想把 data 中的 nan 显示为透明色时，DisplayType 要使用 surface
cb = colorbar('southoutside','fontname','Arial','fontsize',8);
cb.Label.String = 'Label';
print(gcf, 'odiac.png', '-dpng', '-r300'); % 保存为PNG格式，分辨率为 300 dpi
```
![Alt text](/img/Matlab_geoshow/odiac.png)
##  1.3 geoshow 进阶
### 1.3.1 shp 不同元素显示不同的颜色
```
clc;clear
ax = usamap("conus"); % 字体、字号无法像 axesm 在这里进行设置，必须通过 setm 来设置
setm(ax,'fontname','Arial','fontsize',8)
getm(ax)
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
print(gcf, 'usamap.png', '-dpng', '-r300'); % 保存为PNG格式，分辨率为 300 dpi
```
![Alt text](/img/Matlab_geoshow/usamap.png)
### 1.3.2 只显示目标区域的栅格
> geoshow 函数中，我暂时还没有找到现成的白化函数。这里用到了我自己定义的函数[clip_tif](https://github.com/peisipand/matlab_function_by_pzp/blob/main/clip_tif.m),用之前提前下载好。
```
clc;clear;
[data,dataR] = egm96geoid;
states=shaperead("usastatelo.shp",'UseGeoCoords',true);
roi_lat = states(5).Lat;
roi_lon = states(5).Lon;
[clipped_data,clipped_dataR] = clip_tif(data,dataR,roi_lat,roi_lon);
ax = usamap('conus')
setm(ax,'fontname','Arial','fontsize',8)
getm(ax)
geoshow(clipped_data,clipped_dataR,'DisplayType','surface')
geoshow(states,'FaceAlpha',.0) %绘制边界线
print(gcf, 'California.png', '-dpng', '-r300'); % 保存为PNG格式，分辨率为 300 dpi
```

![Alt text](/img/Matlab_geoshow/California.png)

# 二、 m_map工具箱

## 2.1 m_map 绘制 shp (以 OCO2 为例)
```
clc;
clear;
filename = 'F:\硕士\深圳碳中和会议\experiment\data\oco2_LtCO2_180225_B10206Ar_200730003152s.nc4';
lat = ncread(filename,'latitude');
lon = ncread(filename,'longitude');
xco2 = ncread(filename,'xco2');
lon_limit = [118 122];
lat_limit = [38 41];
id = (lat > lat_limit(1)) & (lat < lat_limit(2)) & (lon > lon_limit(1)) & (lon < lon_limit(2));%筛选出研究区内的数据
all = [lat(id,:) lon(id,:) xco2(id,:) ];

figure(1)
set(gcf,'unit','centimeters','position',[25,15,14,11.4]); % 图形窗口在电脑屏幕上的位置和尺寸[左 下 宽 高]
m_proj('mercator', 'longitudes', lon_limit, 'latitudes', lat_limit)
% linestyle 表示是否显示格网
m_grid('box','fancy', 'linestyle', 'none', 'tickdir', 'out','tickstyle','dd','fontname','Arial','fontsize',8);
m_mapshow('E:\MeteoInfo\map\country.shp')
hold on;
m_scatter(all(:,2),all(:,1),30,all(:,3) ,'filled', 'MarkerFaceColor', 'flat', 'MarkerEdgeColor', 'w','linewi',1) ;% lon, lat, 点的大小， 点的颜色映射， 实心
colormap(m_colmap('jet','step',10))
cb = colorbar('southoutside','fontname','Arial','fontsize',8);
cb.Label.String = 'XCO2 (ppm)';
caxis([407 414])
print(gcf, 'oco2.png', '-dpng', '-r300'); % 保存为PNG格式，分辨率为 300 dpi
```

![Alt text](/img/Matlab_geoshow/oco2.png)

## 2.2 m_map 绘制 栅格 (以 PRISMA 为例)

```
clc;clear
filename = 'F:\博士\高光谱甲烷识别\最开始的工作\数据\山西5\PRS_L1_STD_OFFL_20210206031837_20210206031842_0001.he5';
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
% 关于 m_ruler
% 0.7 和 1表示比例尺 起始、终止的位置
% 0.95 表示 垂直方向上的坐标
% 1 表示 比例尺被分为多少份
% linewid 和 ticklen 分别表示 线的宽度和 刻度线的长度
m_ruler([.7 1],.95,1,'linewid',2,'ticklen',[.02],'fontsize',8); % 横向的比例尺
m_grid('box','on', 'tickdir', 'out', 'tickstyle','dd','fontname','Arial','fontsize',8); % 如果觉得默认生成的标签丑，可以用下面的语句自定义 tick
% m_grid('box','on', 'tickdir', 'out', 'tickstyle','dd','fontname','Arial','fontsize',8,...
%     'xtick',112.8:0.1:113.1,...
%     'xticklabel',['112.8';'112.9';'113.0';'113.1'],...
%     'ytick',36.1:0.1:36.4,...
%     'yticklabel',['36.1';'36.2';'36.3';'36.4']);
colormap(m_colmap('jet'));
cb = colorbar('southoutside','fontname','Arial','fontsize',8);
cb.Label.String = 'Label';
caxis([0 10]);
xlabel('Longitude (°E)');
ylabel('Latitude(°N)');
print(gcf, 'prisma.png', '-dpng', '-r300'); % 保存为PNG格式，分辨率为 300 dpi
```
![Alt text](/img/Matlab_geoshow/prisma.png)

## 2.3 m_map 只绘制目标区域的栅格

[bugsuse](http://i-lightning.cn/2020/05/matlab_mask/) 公开的他写的函数，可以轻松实现白化。

# 三、geoshow on GeographicAxes

不同于前文提到的 ``axesm`` 创建的 ax 类型坐标区，``geoaxes`` 创建的是 GeographicAxes 类型坐标区。
* 优点：能显示高分辨的卫星（街道等等）底图
* 缺点：只能显示矢量数据，绘制不了栅格。

下面以OCO-2数据为例进行绘制，数据批量下载的操作可以参考我[之前的博客](https://blog.csdn.net/peisipand/article/details/108224156?spm=1001.2014.3001.5501)。
```
clc;
clear;
filename = 'F:\硕士\深圳碳中和会议\experiment\data\oco2_LtCO2_180225_B10206Ar_200730003152s.nc4';
lat = ncread(filename,'latitude');
lon = ncread(filename,'longitude');
xco2 = ncread(filename,'xco2');

lon_limit = [118 122];
lat_limit = [38 41];
id = (lat > lat_limit(1)) & (lat < lat_limit(2)) & (lon > lon_limit(1)) & (lon < lon_limit(2));%筛选出研究区内的数据
all = [lat(id,:) lon(id,:) xco2(id,:)];
%% 绘制
gx = geoaxes('FontName','Arial','FontSize',8)
geoscatter(all(:,1),all(:,2),2,all(:,3)) % 虽然上一步 设置号了字体和字号，geoscatter一运行，字体又会变成默认的。
caxis([407 414])
geobasemap streets
% geobasemap satelite
geolimits('manual')
geolimits(lat_limit,lon_limit)
gx.LongitudeLabel.String = 'Longitude'
gx.LatitudeLabel.String = 'Latitude'
set(gca,'FontName','Arial','FontSize',8) % 重新定义字体和字号
print(gcf, 'geoaxes.png', '-dpng', '-r300'); % 保存为PNG格式，分辨率为 300 dpi
```
![Alt text](/img/Matlab_geoshow/geoaxes.png)

OCO-2的数据通常以nc和h5的格式存储，h5较于nc，存储了更多的信息，读取h5的代码的代码如下：

```
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
# 补充
其中可以通过load导入的MATLAB自带数据有：

- coastlines - 世界海岸线经纬度矢量

- conus - 用于连接的美国(conus)、五大湖、州际边界的经纬度阵列
- geoid60c - 全球大地水准面高度网格（以米为单位）/度
- greatlakes - 显示结构阵列中的北美五大湖
- korea5c - 朝鲜半岛的地形和水深测量
- koreaEQdata - 地震位置和震级
- layermtx - 用于教学的地理定位地形网格
- mapmtx - 用于教学的地理定位地形网格
- moonalb20c - 克莱门汀全球月球反照率图
- moontopo60c - 月球的克莱门汀激光雷达地形
- oceanlo - 显示结构数组中的海洋遮罩多边形
- russia - 网格化土地、水域、边界、外部区域
- seatempm -全球多通道海面温度网格
- stars - 4500+颗恒星的天体坐标和星等
- usamtx - 美国各州的数据网格，每度五个单元格

usgslulegend - USGS 土地利用类别列表

可以通过shaperead导入的MATLAB自带数据有：

- landareas.shp - 全球陆地区域多边形

- tsunamis.shp - 全球1950-2006 年中到大型海啸的百分比

- usastatehi.shp - 高分辨率多边形美国各州形状

- usastatelo.shp - 低多边形美国各州形状

- worldcities.shp - 全球318个城市或人口稠密位置坐标

- worldlakes.shp - 世界上 37 个最大的多边形湖泊和内陆海域

- worldrivers.shp - 世界主要河流的线条形状

- boston_placenames.shp - 美国马萨诸塞州波士顿地名

- boston_roads.shp - 美国马萨诸塞州波士顿道路

- concord_hydro_area.shp - 美国马萨诸塞州康科德水域

- concord_hydro_line.shp - 美国马萨诸塞州康科德水路

- concord_roads.shp - 美国马萨诸塞州康科德道路

# 参考链接

[Matlab绘制lambert投影下的中国地图，包含南海诸岛小图 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/447264964)
