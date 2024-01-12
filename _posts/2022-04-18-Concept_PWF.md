---
layout:     post
title:      压力加权函数（Pressure weighting function）的计算
subtitle:   
date:       2022-04-18
author:     Peisipand
header-img: img/wallhaven-dg3opm.jpg
catalog: true
tags:
    - Concept
---






---

# 一、为什么需要压力加权函数

以二氧化碳（CO2）为例进行说明。不同高度处的CO2干空气浓度（CO2分子数/总干空气分子数）是不相同的，如果想定义空气柱（地表以上70km）上总的CO2干空气柱浓度$XCO_2$，那么就有

$$
X_{CO_2} = \frac{\int_{0}^{\infty}u(z)N_d(z)dz}{\int_{0}^{\infty}N_d(z)dz}         \tag{1}
$$

这里$u(z)$是在高度$z$处的CO2在干空气中的浓度，$N_d(z)$是在高度$z$处的总的干空气的分子数。

Connor et al. (2008)用压力直接来给每一层的CO2赋予权重，而不是用干空气。O’Dell et al. (2012)在ACOS算法介绍中，定义了如何计算干空气下的CO2柱平均浓度。
改写(recasting)公式1得下式

$$
X_{CO_2} = \frac{\sum\limits_{i=1}^{N-1}\overline{(cu)_i} \Delta p_i} {\sum\limits_{i=1}^{N-1}\overline{c_i} \Delta p_i}   \tag{2}
$$

打公式太累了，具体参看O’Dell et al. (2012) Appendix A中的内容。这里第一次看可能会有点懵，注意理清layers和levels的关系就好了。OCO是提供了20个levels，level 1表示大气层顶，level 20表示地表。这样相当于只有19个layers，也就是文中的 $h_{i}^{'}$只有19个。


# 二、代码

代码中的pwf是官方提供的压力加权函数，hi是我自己算的，基本是一致，细微的差别可能是由于不同高度的重力加速度我均认识是9.8了，但其实个g是会随高度变化而变化的。
```bash
clc;
clear;
filename = 'F:\研究生\主动反演\OCO\利雅得\oco2_L2DiaGL_02623a_141229_B10004r_200115055913.h5';  %oco2_L2DiaGL_25017a_190316_B10004r_200501022851
index = 7000;
specific_humidity_profile_met = h5read(filename,'/RetrievalResults/specific_humidity_profile_met');
specific_humidity_profile_met = double(specific_humidity_profile_met(:,index));

vector_pressure_levels_met = h5read(filename,'/RetrievalResults/vector_pressure_levels_met');
vector_pressure_levels_met = double(vector_pressure_levels_met(:,index));

vector_pressure_levels = h5read(filename,'/RetrievalResults/vector_pressure_levels');
vector_pressure_levels = double(vector_pressure_levels(:,index));

co2_profile = h5read(filename,'/RetrievalResults/co2_profile');
co2_profile = double(co2_profile(:,index));

surface_pressure_fph = h5read(filename,'/RetrievalResults/surface_pressure_fph');
surface_pressure_fph = double(surface_pressure_fph(index));

pwf = h5read(filename,'/RetrievalResults/xco2_pressure_weighting_function');
pwf = double(pwf(:,index));


q = [interp1(vector_pressure_levels_met,specific_humidity_profile_met,vector_pressure_levels(1:19));specific_humidity_profile_met(72)];
c = (1 - q) /  (9.8 * 28.9634);
c_hat = zeros(19,1);
for i = 1:19
c_hat(i,1) = nanmean(c(i:i+1));
end
delta_pi = [diff([vector_pressure_levels])];
hi_pie = c_hat .* delta_pi ./(sum(c_hat .* delta_pi));
hi = zeros(20,1);
hi(1,1) = 1/2 * hi_pie(1);
for i = 2:18
    hi(i,1) = 1/2 * hi_pie(i-1) + 1/2 * hi_pie(i);
end
hi(19,1) = 1/2 * hi_pie(18) + (1 - 1/2) * hi_pie(19);
hi(20,1) = 1/2 * hi_pie(19);
plot([hi pwf])
```
![picture1](/img/Concept_PWF/1.jpg)

# Reference
> 1.The ACOS CO2 retrieval algorithm – Part 1: Description and validation against synthetic observations. Connor et al. (2008)
>
> 2.Orbiting Carbon Observatory: Inverse method and prospective error analysisOrbiting Carbon Observatory: Inverse method and prospective error analysis. O’Dell et al. (2012) 