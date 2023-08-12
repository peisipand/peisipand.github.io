---
layout:     post
title:      根据Plume估计排放速率
subtitle:   
date:       2023-08-12
author:     Peisipand
header-img: img/art-Anaconda-TensorFlow.jpg
catalog: true
tags:
    - WRF
---



根据WRF-LES生成的逐分钟级的Plume，可以用来总结Ueff和U10的关系式，也可以用来模拟给定排放速率下的羽流


---

    % 利用下载的气象文件里的geopotential计算真实高度
    % Requires:
    %     PB  => Base-state pressure (Pa, 3d)
    %     P   => Perturbation pressure (Pa, 3d)
    %     PHB => Base-state geopotential (m^2/s^2, 3d)
    %     PH  => Perturbation geopotential (m^2/s^2, 3d)
    %     T   => Perturbation potential temperature (K, 3d)
    
    clc;clear;
    filename_3h = 'F:\LES\em_les成功\dx=90\auxhist24_d02_0001-01-01_03_00_00';
    PB = ncread(filename_3h,'PB'); % Base-state pressure (Pa, 3d)
    P = ncread(filename_3h,'P'); % Perturbation pressure (Pa, 3d)
    ptot = P + PB;
    T = ncread(filename_3h,'T');
    t = (300+T) .* ( ptot ./ 100000) .^ (0.2854); % kappa = 0.2854.
    PH = ncread(filename_3h,'PH'); % Perturbation geopotential (m^2/s^2, 3d)
    PHB = ncread(filename_3h,'PHB'); % Base-state geopotential (m^2/s^2, 3d)
    height = ( PHB + PH ) ./ 9.81; % compute height according geopotential




# 参考链接

> 1. https://yidongwonyi.wordpress.com/models/wrf-weather-research-and-forecasting/ptot-total-pressure-base-pressure-perturbation-pressure/
> 2. http://gradsusr.org/pipermail/gradsusr/2011-December/031698.html
> 3. https://mailman.ucar.edu/pipermail/wrf-users/2010/001896.html