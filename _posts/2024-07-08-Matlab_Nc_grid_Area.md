---
layout:     post
title:      Matlab函数——用于计算nc经纬度网格对应的面积
subtitle:   A matlab funtion for Calculating grid area based on point lat, point lon and resolution 
date:       2024-07-08
author:     Peisipand
header-img: img/wallhaven-dg3opm.jpg
catalog: true
tags:
    - Matlab
---

当nc文件涉及到单位转换时，比如需要把单位从 kt/month 变成 molec/cm2/s，这时我们就需要知道每个网格的面积。同样的 0.1度*0.1度的网格，在不同纬度时，网格面积也是不一样的。

```
function [output] = area(lat_point,lon_point,res)
% 该函数是计算中心经纬度为(lat_point,lon_point)，分辨率为res的格网面积
% lat_point为输入点的纬度
% lon_point为输入点的经度
% res为分辨率

% distance 函数输出的第一列的距离单位为度，第二列的距离单位为千米
[~,distance1] = distance(lat_point - res / 2,lon_point - res / 2 ,lat_point - res / 2,lon_point + res / 2); % in km
[~,distance2] = distance(lat_point + res / 2,lon_point - res / 2 ,lat_point + res / 2,lon_point + res / 2);% in km 
output = distance1 .* distance2; % in km2
end
```



`area(30,119,0.3)` 输出表示 中心经纬度为 (30,119)，大小为 0.3度 * 0.3度的网格的面积。