---
layout:     post
title:      光谱连续体去除
subtitle:   Continuum Removal
date:       2024-07-08
author:     Peisipand
header-img: img/wallhaven-dg3opm.jpg
catalog: true
tags:
    - Concept
    - Matlab
---

## Google [Continuum Removal]

搜到[ENVI](https://www.nv5geospatialsoftware.com/docs/ContinuumRemoval.html) 关于 Continuum Removal 的解释如下

> Use **Continuum Removal** to normalize reflectance spectra so you can compare individual absorption features from a common baseline. The continuum is a convex hull fit over the top of a spectrum using straight-line segments that connect local spectra maxima. The first and last spectral data values are on the hull; therefore, the first and last bands in the output continuum-removed data file are equal to 1.0.
>
> *Note:* Using different spectral subsets gives different results, so you should spectrally subset the data to the region containing the absorption features of interest.
>
> The continuum is removed by dividing it into the actual spectrum for each pixel in the image.
>

这里看看概念，想要代码不大可能，毕竟ENVI是商业软件，只能在软件里使用该工具包。

## Google [continuum-removed spectral]

搜到论文 [Detection of *Campylobacter* colonies using hyperspectral imaging](https://link.springer.com/article/10.1007/s11694-010-9094-0)

该文章虽然不是大气遥感相关，但详细说明并展示了 Continuum Removal 的结果

> The continuum-removal analysis is aimed to quantify absorption bands depart from a common baseline [16]. The common baseline (i.e., continuum) is defined as the convex hull surrounding the data points of a reflectance spectrum [16–18]. In other words, the continuum consists of piecewise continuous lines connecting local maximum points of the reflectance spectrum (Fig. 5a). Continuum removal is a procedure to isolate a particular absorption feature for analysis by dividing the reflectance spectrum at each wavelength with the continuum at the corresponding wavelength: R CR  = R/C where R CR is the continuum-removed spectrum, R is the reflectance spectrum and C is the continuum (Fig. 5). The continuum-removed spectral values range from 0 to 1. The first and last points in the reflectance spectrum are always local maxima for the continuum. Hence, the first and last points become 1 in the continuum-removed spectrum (Fig. 5b). After continuum removal was applied, the parametric quantification of the absorption band features can be done with the calculation of band depth, bandwidth and wavelength position [19, 20]. As shown in Fig. 5b, the band depth D of each band can be calculated by subtracting the continuum-removed reflectance from 1: D = 1 − R CR [20]. In this study, the slope information of the continuum-removed spectrum was utilized to enhance differences in absorption features of Campylobacter and non-Campylobacter organisms. This slope information was represented by a band ratio (Fig. 5b).

>连续面去除分析旨在量化偏离共同基线的吸收带[16]。共同基线（即连续面）被定义为反射光谱数据点周围的凸壳[16-18]。换句话说，连续体由连接反射光谱局部最大点的片状连续线段组成（图 5a）。连续体去除是通过将每个波长的反射光谱与相应波长的连续体相除，从而分离出特定吸收特征进行分析的过程： R CR = R/C，其中 R CR 是去除的连续谱，R 是反射谱，C 是连续谱（图 5）。反射光谱中的第一个点和最后一个点总是连续面的局部最大值。因此，在去除连续光的光谱中，第一个点和最后一个点都变成了 1（图 5b）。去除连续波后，可以通过计算波段深度、带宽和波长位置来量化吸收波段特征[19, 20]。如图 5b 所示，每个波段的波段深度 D 可通过从 1 中减去去除连续波的反射率来计算：D = 1 - R CR [20]。在本研究中，利用去除连续光光谱的斜率信息来增强弯曲杆菌和非弯曲杆菌吸收特征的差异。

![图1](\img\Matlab_Continuum removal\1.png)

## Google [continuum removal matlab]

搜到 Matlab 2024 提供了相关的工具包

[Normalize spectral signature - MATLAB removeContinuum - MathWorks 中国](https://ww2.mathworks.cn/help/images/ref/removecontinuum.html)

