---
layout:     post
title:      Matlab对栅格数据重采样
subtitle:   Resample raster data using Matlab 
date:       2024-07-08
author:     Peisipand
header-img: img/wallhaven-dg3opm.jpg
catalog: true
tags:
    - Matlab
---

栅格数据经常会涉及到重采样，如将栅格A的分辨率重采样成栅格B的分辨率。如果是tif格式，**ENVI** 的 `resample` 工具包可以轻松完成这个任务。

有时我们需要用Matlab读取nc文件，如果 nc转tif，再去ENVI裁剪显得有些麻烦了。本文提供了matlab的样例程序，以降低（网格平均）或提高（线性插值）数据的分辨率。

```
% 从高分辨到低分辨
% --- 低分辨率 0.25°
deg = 0.25;                            
lat_era = 16:deg:47.75;                   
lon_era = 96:deg:127.75;
% --- 原始数据的高分辨率的经纬度数据
lat = ncread(full_path,'lat'); 
lon = ncread(full_path,'lon'); 
for ii=1:length(lat_era)
	for jj=1:length(lon_era)
			ix1 = find((lat_era(ii)-deg/2<=lat & lat<=lat_era(ii)+deg/2));
			ix2 = find((lon_era(jj)-deg/2<=lon & lon<=lon_era(jj)+deg/2));
			if ~isempty(ix1) & ~isempty(ix2)
				t2(jj,ii) = nanmean(nanmean(T2(ix2,ix1)));
			end
	end
end

```

> 注意：重采样时一定要看变量的单位：
>
> 如果变量是 温度(K)、气体浓度(ppm)，那么从细网格到粗网格是求平均的关系。
>
> 如果变量是 通量(kg/grid)这种，那么从细网格到粗网格是求和的关系。



---

参考链接

[地理数据中的分辨率转换_如何2.5*2.5提高成1*1分辨率气象数据-CSDN博客](https://blog.csdn.net/qq_38734327/article/details/129397372)