---
layout:     post
title:      使用 MATLAB 绘制中国各省的煤炭产量和山西的煤矿分布
subtitle:   Plotting Coal Production in Chinese Provinces and Coal Mining Distribution in Shanxi Using MATLAB
date:       2024-06-18
author:     Peisipand
header-img: img/wallhaven-dg3opm.jpg
catalog: true
tags:
    - Matlab
    - Plot
---


---

这里记录并分享下我在写上一篇论文([Unveiling unprecedented methane hotspots in China's leading coal production hub: A satellite mapping revelation](https://zhipengpei.top/publication/han-2024-unveiling/))时作图的代码。效果如图
![picture1](/img/Matlab_plot_coal_mines/Fig1.jpg "图1")
左边这个图上有两个图层，分别是各省的煤炭产量（来自能源局）及煤矿甲烷排放（来自EDGAR清单），并用了不同的colormap，为了简单起见，我是分别绘制了两个图层，然后把两个图的colorbar抠出来再放到一个图的。关于geoshow的使用，可以参考[之前的博客](https://peisipand.github.io/2022/04/13/Matlab_geoshow/)。

```
F:\博士\山西煤矿调查\代码\plot_Fig1.m
clc;clear
china = shaperead('F:\博士\山西煤矿调查\数据\shp\cn_province.shp','UseGeoCoords',true);
coalData = [china.Coal_prod] / 1e3; % 提取煤炭产量数据,原本的单位是万吨
max_data = max(coalData); %最大产量
k= 6;
mycolormap=summer(k);
mysymbolspec=cell(1,36); % 预定义变量可以加快处理速度
for i=1:36
    i_coalData = coalData(i) ; % 第i个省的产量
    if i_coalData < 5 
        mycoloridx = 1;
    end
    if i_coalData < 10 && i_coalData > 5
        mycoloridx = 2;
    end
    if i_coalData < 20 && i_coalData > 10
        mycoloridx = 3;
    end
    if i_coalData < 40 && i_coalData > 20
        mycoloridx = 4;
    end
    if i_coalData < 80 && i_coalData > 40
        mycoloridx = 5;
    end
    if i_coalData > 80
        mycoloridx = 6;
    end
    mysymbolspec{i} = {'Coal_prod', i_coalData * 1e3, 'FaceColor', mycolormap( mycoloridx, :) };
    if i_coalData < 0
        mysymbolspec{i} = {'Coal_prod', i_coalData, 'FaceColor', 'none' };
    end
end
% 最关键的两个语句
symbols=makesymbolspec('Polygon',{'default','FaceColor','none',...
    'LineStyle','--','LineWidth',0.2,...
    'EdgeColor',[0.8 0.9 0.9]},...
    mysymbolspec{:}...
    );
Proj='mercator'; %投影方式
figure;
set(gcf,'unit','centimeters','position',[25,15,12,8]); % 图形窗口在电脑屏幕上的位置和尺寸[左 下 宽 高]
%设置图像大小
latmin=16; latmax=55; lonmin=70; lonmax= 140;
% 设置显示范围 也可以通过后面的 setm(ax1,'MLineLocation',10)
% Frame 外边框；flinewidth 外边框粗细；Grid 里面的虚线；mlabel横；plabel纵;axis关
ax1=axesm(Proj,'MapLatLimit',[latmin latmax], 'MapLonLimit', [lonmin lonmax],...
    'Frame','on','flinewidth',2,'Grid','on', 'FontName', 'Arial', 'FontSize', 8,...
    'MLineLocation',10,'PLineLocation',10,'ParallelLabel','on','MeridianLabel','on','mlabelparallel','south');
geoshow(ax1,china,'SymbolSpec',symbols); % 此处用mapshow投影会不正确
% colormap(summer(k))
% cb=colorbar('southOutside');
% set(cb,'YTick',0:1/6:1);  % 默认的范围是 0-1，只能平分6份了
% set(cb,'YTickLabel',num2cell([0 5 10 20 40 80 120]))
% set(cb,'FontName','Arial'); 
axis off; %不加这段代码的话，图就太丑了，轴的刻度线就超过了画图范围
hold on;
cn_province=shaperead('F:\博士\山西煤矿调查\数据\shp\cn_province.shp','UseGeoCoords',true); % polygon
geoshow(cn_province,'facecolor','none','FaceAlpha',1,'Edgecolor','black');%这里
file_name = 'F:\博士\山西煤矿调查\数据\EDGARv8\v8.0_FT2022_GHG_CH4_2022_PRO_COAL_emi.nc';
data = ncread(file_name,'emissions'); % in Tonnes
data = double(data');
data(data==0) = nan;
[lat_steps,lon_steps] = size(data);
dataR = georasterref('RasterSize',[lat_steps lon_steps], 'LatitudeLimits', [-89.95 89.95], 'LongitudeLimits', [ -179.95 179.95]);
geoshow(data,dataR ,'DisplayType', 'surface');%这里
colormap(brewermap(256,'YlOrRd')) % YlOrRd
% hcb=colorbar('southOutside','FontName', 'Arial', 'FontSize', 8);
caxis([0,1e5]) % colorbar的范围
set(gca,'looseInset',[0 0 0 0])
print(gcf, 'china-coal&edgar.png', '-dpng', '-r300'); % 保存为PNG格式，分辨率为300
```




绘制右边这个图，即山西的煤矿位置+卫星的过境情况
```
%% 制图 山西省  过境位置+煤矿位置
clc;clear;
Proj='mercator'; %投影方式
figure;
set(gcf,'unit','centimeters','position',[25,15,7,11.4]); % 图形窗口在电脑屏幕上的位置和尺寸[左 下 宽 高]
%设置图像大小
latmin=34.3 ; latmax=41; lonmin=110; lonmax= 114.7;
% 设置显示范围 也可以通过后面的 setm(ax1,'MLineLocation',10)
% Frame 外边框；flinewidth 外边框粗细；Grid 里面的虚线；mlabel横；plabel纵;axis关
ax1=axesm(Proj,'MapLatLimit',[latmin latmax], 'MapLonLimit', [lonmin lonmax],...
    'Frame','on','flinewidth',2,'Grid','on', 'FontName', 'Times', 'FontSize', 8,...
    'MLineLocation',2,'PLineLocation',2,'ParallelLabel','on','MeridianLabel','on','mlabelparallel','south');
axis off; %不加这段代码的话，图就太丑了，轴的刻度线就超过了画图范围
shp_boundry=shaperead('Step3_排放速率估计/shp_boundry.shp','UseGeoCoords',true); % polygon
geoshow(shp_boundry,'facecolor',[255,235,190] / 255,'FaceAlpha',0.1,'Edgecolor',[0.25,0.25,0.25]);%这里
shanxi_city = shaperead('F:\博士\山西煤矿调查\数据\shp\sx_cities.shp','UseGeoCoords',true); % polygon
geoshow(shanxi_city,'facecolor','none','FaceAlpha',1,'Edgecolor',[0,0,0]);%这里
shanxi_mines = shaperead('F:\博士\山西煤矿调查\数据\山西能源局\mines_689.shp','UseGeoCoords',true); % point
geoshow(shanxi_mines,'DisplayType','point','Marker','.','Color','red','MarkerSize',2);%这里
set(gca,'looseInset',[0 0 0 0])
print(gcf, 'shanxi.png', '-dpng', '-r300'); % 保存为PNG格式，分辨率为300
```


