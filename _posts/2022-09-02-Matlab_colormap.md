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
以 gray 为例，输入 ``gray`` 会返回三列数据，分别对应RGB。

```bash
data = peaks(5)
imagesc(data)
cm = gray;
colormap(cm)
```
0 0 0 代表黑色，cm 从上之下对应从黑色到白色。原理搞清楚之后，我们就可以DIY色带了。下面了参考一些审美在线的配色。

# 一、NCL

[NCL官网](https://www.ncl.ucar.edu/Document/Graphics/color_table_gallery.shtml)提供了很多颜色图合集，如下所示
![picture1](/img/Matlab_colormap/ncl.png "图1")

以下是我写的自动从网上获取ncl colormap RGB值的函数，该函数也可以从[我的github](https://github.com/peisipand/matlab_function_by_pzp/blob/main/ncl_colormap.m)中获取。

```bash
function color = ncl_colormap(colorname)

% colorname 可以从下面这个网站中获取：
% https://www.ncl.ucar.edu/Document/Graphics/color_table_gallery.shtml
% 示例如下：
% cm = ncl_colormap('cmocean_deep')
% colormap(cm)
% cb = colorbar()

% Author: Zhipeng Pei (peisipand@whu.edu.cn)
% Date: 2024-02-02

url = ['https://www.ncl.ucar.edu/Document/Graphics/ColorTables/Files/',colorname,'.rgb'];
sourcefile=urlread(url,'get','');
source = strtrim(sourcefile);
source = regexp(source, '\s+', 'split');
index_r = find(strcmp(source, 'r'));
index_g = find(strcmp(source, 'g'));
index_b = find(strcmp(source, 'b'));
if (index_b == index_g +1) && (index_g == index_r +1)
    source(1:index_b) = [];
end
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
data = peaks(5)
imagesc(data)
cm = ncl_colormap('cmocean_balance');
colormap(cm)
caxis([-1 1])
```

![picture1](/img/MATLAB colormap/ncl_cmocean_balance.jpg "图2")

# 二、BREWERMAP

[colorBrewer](https://colorbrewer2.org/#type=sequential&scheme=BuGn&n=3)提供了很多配色方案，MATLAB中，[BREWERMAP](https://www.mathworks.com/matlabcentral/fileexchange/45208-colorbrewer-attractive-and-distinctive-colormaps) 、 [cbrewer2](https://www.mathworks.com/matlabcentral/fileexchange/58350-cbrewer2) 工具包提供了所有 ColorBrewer 配色方案。俩个函数异曲同工，这里以BREWERMAP为例介绍。

![picture1](/img/Matlab_colormap/brewer_schem.png "图3")

安装完成后，调用函数即可查看。

```bash
brewermap_plot
```
![picture1](/img/Matlab_colormap/brewermap_plot.jpg "图4")

```bash
brewermap_view
```
![picture1](/img/Matlab_colormap/brewermap_view.jpg "图5")

```
brewermap(N,scheme)
```

N为colormap被插值成多少份，scheme为颜色方案名。

````
brewermap(Nan,'-Blues')
````

``-``表示颜色带反转

```bash
imagesc(column)
colormap(brewermap(256,'*YlGnBu'));
caxis([0 200])
colorbar
```
![picture1](/img/Matlab_colormap/brewermap_YlGnBu.jpg "图6")

# 三、尖角 colorbar

[cbarrow](https://ww2.mathworks.cn/matlabcentral/fileexchange/52515-cbarrow-pointy-ends-for-colorbars) 提供了绘制尖角colorbar的函数。

![picture1](/img/Matlab_colormap/cbarrow.jpg "图7")


# 参考链接

https://blog.csdn.net/wokaowokaowokao12345/article/details/112969927

https://blog.csdn.net/qq_27984679/article/details/104170418