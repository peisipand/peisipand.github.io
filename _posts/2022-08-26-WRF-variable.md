---
layout:     post
title:      WRF变量
subtitle:   
date:       2022-09-02
author:     Peisipand
header-img: img/art-Anaconda-TensorFlow.jpg
catalog: true
tags:
    - WRF
---






---

    Requires:
        PB  => Base-state pressure (Pa, 3d)
        P   => Perturbation pressure (Pa, 3d)
        PHB => Base-state geopotential (m^2/s^2, 3d)
        PH  => Perturbation geopotential (m^2/s^2, 3d)
        T   => Perturbation potential temperature (K, 3d)
    
    # Quick conversions to full pressure and actual temperature
    ptot = P + PB
    t = (300+T) * (ptot/1000.) ** (0.2854)
    # compute geopotential
    ph = (PHB + PH)/9.81
(PH + PHB)/9.8 将为您提供每个模型级别的真实高度

https://yidongwonyi.wordpress.com/models/wrf-weather-research-and-forecasting/ptot-total-pressure-base-pressure-perturbation-pressure/

http://gradsusr.org/pipermail/gradsusr/2011-December/031698.html

https://mailman.ucar.edu/pipermail/wrf-users/2010/001896.html