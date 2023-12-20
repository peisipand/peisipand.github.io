---
layout:     post
title:      Matlab 自定义Legend
subtitle:   
date:       2023-09-13
author:     Peisipand
header-img: img/art-Anaconda-TensorFlow.jpg
catalog: true
tags:
    - Matlab
    - Plot
---


---

起初是看到Sci. Adv.上一篇论文里的一张图挺好看的，想模仿画一下。
![picture1](img/Matlab_legend_DIY/SA.png "SA")

当然，如果作图量不大的话，也可以用Arcgis来出图，这篇博客主要介绍用MATLAB如何来实现绘制这种图。

首先要用高分辨率卫星作为底图，所以我们采用了geobasemap satelite来实现。

用Marker size来映射排放量的大小，这个可以通过geoscatter或者geoplot来实现（类似于scatter和plot）。但当我尝试生成Legend时发现了问题。不管图中的Markers多大，Legend里的Markers的大小与图中的不一致。用中文搜这个问题还搜不到什么可用的解决方案，最后一顿英文搜才搜到合适的方案

[链接1](https://www.mathworks.com/matlabcentral/answers/97779-why-is-the-marker-size-in-the-legend-of-a-scatter-plot-not-equal-to-the-marker-size-in-the-plot)、[链接2](https://ww2.mathworks.cn/matlabcentral/answers/503622-controlling-the-size-of-legend-markers-separately-from-the-font-size-of-legend-labels)

> 总结的解决方案
>
> 1. 如果用scatter的话，可以在生成Legend后再去调整Marker size（需要注意的是scatter函数里的marker size 是Legend里的marker size的平方。
>
> 2. 也可以使用plot来替换scatter，这样生成的Legend里的Marker size和图中就一致了。（需要注意的是这个方案只适合于marker size小于12的情况）

使用scatter函数测试一下

```bash
figure; hold on
s1 = scatter(1, 1, 360, 'k', 'o') 
s2 = scatter(1, 2, 100, 'k', '+')
% [a,b,c] = legend({'one','two'}) % 分别返回 legend group scatter类型，其中Group 中有可以设置Legend里marker的选项
[lgd, objh] = legend({'one','two'}); 
lgd.Position = [0.7577,0.6139,0.1286,0.2]; % x y width height
objhl = findobj(objh, 'type', 'patch');
set(objhl(1), 'Markersize', sqrt(360));
set(objhl(2), 'Markersize', sqrt(100));
axis([0 3 0 3])
```
![picture1](img/Matlab_legend_DIY/scatter test.png "scatter test")

这里加上了对lgd position的控制，是因为如果marker size过大时，Legend的行间距并不会跟着变大，就会导致多个marker挤在一起，为了避免这种情况，增加Legend的height即可。

搞清楚基本原理后，下面使用geoscatter就顺畅多了

```
%% 制图 聚焦 测试
set(gcf,'unit','centimeters','position',[10,10,9,12]) % 图形窗口在电脑屏幕上的位置和尺寸[左 下 宽 高]
lon_limit = [112.818113 113.065933];
lat_limit = [36.212595 36.557130];
gx = geoaxes
% set(gca,'FontName','Times New Roman','FontSize',20) 
% 从colorbrew找的浅黄到棕色的色带
colormap_yellow_to_brown = ...
[254,240,217
253,204,138
252,141,89
227,74,51
179,0,0] ./ 255;
% colormap_yellow_to_brown = reshape(colormap_yellow_to_brown,)
geoscatter(36.25,112.9,100,"square","MarkerEdgeColor",[0 0 0],"MarkerFaceColor",colormap_yellow_to_brown(1,:),"LineWidth",1)
hold on 
geoscatter(36.3,112.9,200,"square","MarkerEdgeColor",[0 0 0],"MarkerFaceColor",colormap_yellow_to_brown(2,:),"LineWidth",1)
geoscatter(36.35,112.9,300,"square","MarkerEdgeColor",[0 0 0],"MarkerFaceColor",colormap_yellow_to_brown(3,:),"LineWidth",1)
geoscatter(36.4,112.9,400,"square","MarkerEdgeColor",[0 0 0],"MarkerFaceColor",colormap_yellow_to_brown(4,:),"LineWidth",1)
geoscatter(36.45,112.9,500,"square","MarkerEdgeColor",[0 0 0],"MarkerFaceColor",colormap_yellow_to_brown(5,:),"LineWidth",1)

geobasemap satellite % satellite  streets-light
geotickformat dd % 用decimal的格式显示经纬度，默认的是度分秒
gx.Scalebar.Visible = "on";
gx.Grid = 'off'; % 网格线关掉
% geobasemap satelite
geolimits(lat_limit,lon_limit)
gx.LatitudeLabel.String = '';
gx.LongitudeLabel.String = '';
[lgd, objh] = legend({'one','two','three','four','five'}); % Instead of "h_legend" use "[~, objh]"
lgd.Position = [0.6577,0.6139,0.1286,0.28]; % x y width height
lgd.FontName = 'Arial';
lgd.FontSize = 8;
objhl = findobj(objh, 'type', 'patch');
set(objhl(1), 'Markersize', sqrt(100));
set(objhl(2), 'Markersize', sqrt(200));
set(objhl(3), 'Markersize', sqrt(300));
set(objhl(4), 'Markersize', sqrt(400));
set(objhl(5), 'Markersize', sqrt(500));
```

![picture1](img/Matlab_legend_DIY/geoscatter test.png "geoscatter test")





# 一、NCL

[NCL官网](https://blog.csdn.net/peisipand?spm=1001.2101.3001.5343)提供了很多颜色图合集，如下所示
![picture1](img/Matlab_legend_DIY/ncl.png "图1")

以下是我写的自动从网上获取ncl colormap RGB值的函数。

```bash
function color = ncl_colormap(colorname)
url = ['https://www.ncl.ucar.edu/Document/Graphics/ColorTables/Files/',colorname,'.rgb'];
sourcefile=urlread(url,'get','');
source = strtrim(sourcefile);
source = regexp(source, '\s+', 'split');
source(1:7) = [];
for i=1:(size(source,2))
    color(i) = str2double(source{i});    
end
integer = round(size(color,2) ./ 3); %这里是为了防止个数 不能把被3整除
color = color(1,1:integer * 3);
color = reshape(color,3,size(color,2) / 3);
color = color';
color = single(color) ./ 255;
end
```

以'cmocean_balance'为例，调用方式如下

```bash
imagesc(column)
colormap(ncl_colormap('cmocean_balance'))
caxis([0 200])
```

![picture1](/img/Matlab_legend_DIY/ncl_cmocean_balance.jpg "图2")

# 二、BREWERMAP

[colorBrewer](https://colorbrewer2.org/#type=sequential&scheme=BuGn&n=3)提供了很多配色方案，MATLAB中，[BREWERMAP](https://www.mathworks.com/matlabcentral/fileexchange/45208-colorbrewer-attractive-and-distinctive-colormaps)工具包提供了所有 ColorBrewer 配色方案。

安装完成后，调用函数即可查看。

```bash
brewermap_plot
```
![picture1](/img/Matlab_legend_DIY/brewermap_plot.jpg "图3")

```bash
brewermap_view
```
![picture1](/img/Matlab_legend_DIY/brewermap_view.jpg "图4")

```bash
imagesc(column)
colormap(brewermap(256,'*YlGnBu'));
caxis([0 200])
colorbar
```
![picture1](/img/Matlab_legend_DIY/brewermap_YlGnBu.jpg "图5")




# 参考链接

https://blog.csdn.net/wokaowokaowokao12345/article/details/112969927

https://blog.csdn.net/qq_27984679/article/details/104170418