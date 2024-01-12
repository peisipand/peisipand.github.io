---
layout:     post
title:      matlab colormap
subtitle:   
date:       2022-09-02
author:     Peisipand
header-img: img/wallhaven-dg3opm.jpg
catalog: true
tags:
    - Matlab
    - Plot
---


---

MATLAB提供了几个基础的colormap，例如gray, cool, hot, parula(默认的)等。
实现的原理如下，以gray为例，输入gray会返回三列数据，分别对应RGB，从上之下对应从白色到黑色。

```bash
imagesc(column)
colormap(gray)
```
原理搞清楚之后，我们就可以DIY色带了。下面了参考一些审美在线的配色。

# 一、NCL

[NCL官网](https://www.ncl.ucar.edu/Document/Graphics/color_table_gallery.shtml)提供了很多颜色图合集，如下所示
![picture1](/img/Matlab_colormap/ncl.png "图1")

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

![picture1](/img/MATLAB colormap/ncl_cmocean_balance.jpg "图2")

# 二、BREWERMAP

[colorBrewer](https://colorbrewer2.org/#type=sequential&scheme=BuGn&n=3)提供了很多配色方案，MATLAB中，[BREWERMAP](https://www.mathworks.com/matlabcentral/fileexchange/45208-colorbrewer-attractive-and-distinctive-colormaps)工具包提供了所有 ColorBrewer 配色方案。

安装完成后，调用函数即可查看。

```bash
brewermap_plot
```
![picture1](/img/Matlab_colormap/brewermap_plot.jpg "图3")

```bash
brewermap_view
```
![picture1](/img/Matlab_colormap/brewermap_view.jpg "图4")

```bash
imagesc(column)
colormap(brewermap(256,'*YlGnBu'));
caxis([0 200])
colorbar
```
![picture1](/img/Matlab_colormap/brewermap_YlGnBu.jpg "图5")




# 参考链接

https://blog.csdn.net/wokaowokaowokao12345/article/details/112969927

https://blog.csdn.net/qq_27984679/article/details/104170418