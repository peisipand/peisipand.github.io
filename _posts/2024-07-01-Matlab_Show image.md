---
layout:     post
title:      使用 MATLAB 绘制高光谱数据的RGB影像
subtitle:   Plotting RGB images of hyperspectral data using MATLAB
date:       2024-07-01
author:     Peisipand
header-img: img/wallhaven-dg3opm.jpg
catalog: true
tags:
    - Matlab
    - Plot
---


---

# 前言

我们知道，任何一个颜色（即相机或人眼记录的颜色），都可以用RGB（R、G、B分别为 0至255之间的一个数字）来表示，比如 (255, 255, 255) 代表着白色；(255, 0, 0) 代表着红色。相机的CMOS传感器有着交叉排列的 红、绿、蓝 三色传感器，所以可以记录这三个波长的信号大小，最终以照片的形式来呈现。

高光谱成像仪不同于普通的相机，它可以记录 400-2500nm范围内很多个波长的信号，而非仅仅 红绿蓝三个波段。如果我们正确的将 红光波段、绿光波段和蓝光波段当做 RGB 的三个值，那么得到的就是真彩色图像，如果故意错误的将 某三个波段赋予 RGB，那么我们就可以得到一个假彩色图像了。

如果只展示高光谱数据中的其中一个波段（行*列的2D矩阵），就相当于要给所有的数字索引到一个颜色上。

# 一、二维矩阵的映射

对于二维矩阵而言，绘制它本质上是先将这个矩阵里的像素值从小到大排列，然后从小到大依次映射的一个 colormap 上，colormap 是提前设置好的一个颜色带。下面来看 `image` ， `imagesc` 和 `imshow`这三个最常用的函数，下面以经典的 lena 图片进行展示说明。
```
clear all;clc;close all;
img = imread('F:\matlab\lena.jpg');
I = rgb2gray(img);
% Attention: we use the axis off to get rid of the axis.
subplot(2,3,1); image(I); % equals to imagesc(I,[0 255])
colorbar,title('I show by image');axis off;
subplot(2,3,2); imagesc(I); % equals to imagesc(I,[min(I(:)) max(I(:))])
colorbar,title('I show by imagesc');axis off;
subplot(2,3,3); imshow(I); % equals to imshow(I,[0 255])
colorbar,title('I show by imshow');axis off;
subplot(2,3,4); image(I./4); % equals to imagesc(I,[0 255])
colorbar,title('I/4 show by image');axis off;
subplot(2,3,5); imagesc(I./4); % equals to imagesc(I,[min(I(:)) max(I(:))])
colorbar,title('I/4 show by imagesc');axis off;
subplot(2,3,6); imshow(I./4); % equals to imshow(I,[0 255])
colorbar,title('I/4 show by imshow');axis off;
```

![picture1](/img/Matlab_image/1.png "图1")

看第一列，发现把 I 缩小四倍，image 画出来的效果变了，这是由于 colormap 的最大、最小值没变，仍然是 0-255。

看第二列，发现把 I 缩小四倍，imagesc 画出来的效果没变，这是由于 colormap 的最大、最小值是根据数据的情况自动调整的。

看第三列，发现和第一列的区别就是 colormap 从 jet 变成了 gray 。另外限制了纵横比。

小结：

**image**：将double型数据取整（正数取整就是把小数部分舍掉），显示范围为[0 255]。

**imagesc**：这个函数很好，会把显示范围自动设置成[min(I(:)) max(I(:))]。

**imshow**：这个函数调用方式不同，显示效果也不同，如下：

**imshow(I)**：当图像为double型时，imshow 函数会把显示范围设置成[0 , 1],这样小于0的就变成黑色了，大于1的就变成白色了，所以处理不当就会出现全白的情况。当图像为 unit8 型时，imshow 函数会把显示范围设置成[0 , 255]。

**imshow( I/(max(I(:)))**：针对直接调用imshow函数出现的问题，用max(I(:) ) 对图像矩阵进行归一化再显示，这样负数部分会变黑，正数部分还可以正常显示，但有一部分信息丢失了。

**imshow(uint8(I))**：这种方式把I转化成uint8类型，负数会被归零，超过255的被置为255，而且小数也会被round(四舍五入)，当参数为uint8型时，imshow函数把显示范围设置成[0,255]，这样图像虽然也能显示出来，但与原始数据相比来说，丢掉很多信息，但有时可能却是想要的结果，这个要看具体情况。

**imshow(I,[ ])**：这种方式就是把imshow的显示范围设置成[min(I(:)) max(I(:))]，也就是线性映射，相当于imagesc(I),colormap(gray(255))可以将整幅图像的信息显示出来。

# 二、RGB 三维矩阵的显示

 imagesc 在官网介绍中并未提及RGB 矩阵的显示，但是通过它的注释发现仍然该功能。注释如下

```
When C is a 3-dimensional m-by-n-by-3 matrix, the
%   elements in C(:,:,1) are interpreted as red intensities, in C(:,:,2) as
%   green intensities, and in C(:,:,3) as blue intensities, and the
%   CDataMapping property of image is ignored.  For matrices containing
%   doubles, color intensities are on the range [0.0, 1.0].  For uint8 and
%   uint16 matrices, color intensities are on the range [0, 255].
```

MATLAB帮助中心提示如果使用 `imagesc` 函数将图像解释为真彩色 (RGB) 图像，建议使用 [`rescale`](https://ww2.mathworks.cn/help/matlab/ref/rescale.html) 函数缩放真彩色像素值，当然也可以采用[别的方式](https://blog.csdn.net/NBDwo/article/details/115564548)进行归一化。

虽然能正常显示，但是我们在绘制RGB图像时，希望纵横比是固定的，所以还是用 `imshow` 稳妥。

```
function [rgb] = func_hyperImshow(hyper_data,RGBbands)
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
% Author: Zhipeng Pei
% Time: 2024-07-01
% Function Usage
% Input:
%    hyper_data — the 3D hyperspectral dataset with the size of rows x cols x bands
%    RGBbands— the RGB bands to be displayed, with the format [Red Green Blue]
% Output:
%    rgb- the finual result with the RGB bands with the size of (rows x cols x 3)  
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
hyper_data = double(hyper_data);
[rows, cols, bands] = size(hyper_data);

% 归一化
hyper_data = rescale(hyper_data);

[rows, cols, bands] = size(hyper_data);

if (nargin == 1)
    RGBbands = [bands round(bands/2) 1];
end

if (bands ==1)
    red = hyper_data(:,:);
    green = hyper_data(:,:);
    blue = hyper_data(:,:);
else
    red = hyper_data(:,:,RGBbands(1));
    green = hyper_data(:,:,RGBbands(2));
    blue = hyper_data(:,:,RGBbands(3));
end

rgb = zeros(size(hyper_data, 1), size(hyper_data, 2), 3);
rgb(:,:,1) = adapthisteq(red);      % Adaptive histogram equalization
rgb(:,:,2) = adapthisteq(green);
rgb(:,:,3) = adapthisteq(blue);
imshow(rgb); axis image;   % 保持图像的显示比例，其中axis为坐标轴的控制函数。
end
```

效果如果：

![picture1](/img/Matlab_image/2.png "图2")



本文提到的绘图并不能按照实际经纬度展示，如需**结合经纬度绘制**，请参考[之前的博客](https://peisipand.github.io/2022/04/13/Matlab_geoshow/)。
