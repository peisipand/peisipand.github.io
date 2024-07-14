---
layout:     post
title:      辐射传输模型中涉及到的一些概念
subtitle:   Some concepts involved in Radiative Transfer Model (RTM)
date:       2024-07-08
author:     Peisipand
header-img: img/wallhaven-dg3opm.jpg
catalog: true
tags:
    - Concept
---

## 一、表观反射率和地表反射率

>  表观反射率: apparent reflectance
>
> 地表反射率: surface reflectance

当卫星在大气层顶测量到 radiance 之后，我们可以根据太阳初始的 irradiance 来计算大气层顶的 **表观反射率**。有的研究需要关注地表的特性，即需要将 **表观反射率** 转换为 **地表反射率**，下面介绍了具体如何转换。

The apparent reflectance $\rho^*\left(\theta_s, \varphi_s, \theta_v, \varphi_v, \lambda\right)$ is used in the formation of the radiative transfer problem. The definition of **apparent reflectance** is
$$
\rho^*\left(\theta_s, \varphi_s, \theta_v, \varphi_v, \lambda\right)=\frac{\pi L\left(\theta_s, \varphi_s, \theta_v, \varphi_v, \lambda\right)}{\mu_s E_s(\lambda)},(1)
$$

where $\theta_s$ is the solar zenith angle, $\varphi_s$ is the solar azimuth angle, $\theta_v$ is the sensor zenith angle,  $\varphi_v$ is the sensor azimuth angle, $\lambda$ is wavelength, $L$ is the radiance measured at the satellite, $E_s$ is the solar flux at the top of the atmosphere (TOA), and $\mu_s = cos \theta_s$. According to Tanre et al. (1986), for a horizontal surface of uniform Lambertian reflectance, $\rho^*\left(\theta_s, \varphi_s, \theta_v, \varphi_v, \lambda\right)$ can be expressed as

$$
\begin{gathered}\rho^*\left(\theta_s, \varphi_s, \theta_v, \varphi_v, \lambda\right)=T_g\left(\theta_s, \theta_v, \lambda\right) \\ \times\left[\rho_a\left(\theta_s, \varphi_s, \theta_v, \varphi_v, \lambda\right)+\frac{T\left(\theta_s, \lambda\right) T\left(\theta_v, \lambda\right) \rho(\lambda)}{1-\rho(\lambda) S(\lambda)}\right]\end{gathered},(2)
$$

where $T_g$ is the total gaseous transmittance in the Sun-surface-sensor path, $\rho_a$ is the atmospheric reflectance, which is related to the path radiance resulted from atmospheric scattering, $T\left(\theta_s\right)$ is the downward scattering transmittance, $T\left(\theta_v\right)$  is the upward scattering transmittance, $S$ is the spherical albedo of the atmosphere, and $\rho$ is the **surface refectance**. $T_g$ is expressed as

$$
T_g\left(\theta_s, \theta_v, \lambda\right)=\prod_{i=1}^n T_{g i}\left(\theta_s, \theta_i, \lambda\right),(3)
$$

where $T_{gi}$  is the transmittance of the $i$th gas in the Sun-surface-sensor path and $n$ is the number of gases. $T_g$ is calculated by assuming that there is no atmospheric scattering. The transmittances in Eq. (3) refer to the average transmittances over narrow spectral intervals (a few nanometers).

The scattering terms, $\rho$, $T(\theta_s)$, $T(\theta_s)$, and $S$, are calculated by assuming that there are no atmospheric gaseous absorptions. In the real atmosphere, the scattering and absorption processes occur simultaneously. Equation (2) treats these as two independent processes. The coupling effects between the two processes are neglected. The coupling effects are small in regions where the atmospheric gaseous absorptions are weak and in regions where the scattering effects are small. Equation (3) is strictly correct only in regions where there are no overlapping absorptions by different gases. It remains a good approximation for the 0.4-2.5 $\mu$m region because little overlapping absorptions occur in this region. 

The 5S code was originally designed to simulate radiances measured from satellite platforms using geometric and atmospheric models and using surface reflectances as boundary conditions. However, the code can be modified for retrieving surface reflectances from measured radiances (Teillet, 1989). Solving Eq. (2) for $\rho$ yields

$$
\begin{aligned} \rho(\lambda)= & \left(\frac{\rho^*\left(\theta_s, \varphi_s, \theta_v, \lambda\right)}{T_g\left(\theta_s, \theta_v, \lambda\right)}-\rho_a\left(\theta_s, \varphi_s, \theta_v, \varphi_v, \lambda\right)\right) \\ & \times\left[ T\left(\theta_s, \lambda\right) T\left(\theta_v, \lambda\right) + S(\lambda)\left(\frac{\rho^*\left(\theta_s, \varphi_s, \theta_v, \varphi_v, \lambda\right)}{T_g\left(\theta_s, \theta_v, \lambda\right)} -\rho_a\left(\theta_s, \varphi_s, \theta_v, \varphi_v, \lambda\right)\right) \right]^{-1} \end{aligned}, (4)
$$

Given a satellite measured radiance, the surface reflectance can be derived according to Eqs. (1) and (4).

**Reference:**

*Derivation of Scaled Surface Reflectances from AVIRIS Data, Bo-Cai Gao et al, 1993, RSE.*  

## 二、地表反照率和地表反射率

> 地表反照率: surface albedo
>
> 地表反射率: surface reflectance

**关于地表反照率的定义：**

- [维基百科](https://en.wikipedia.org/wiki/Albedo): 反照率是一个物体漫反射太阳光的比例。其测量范围从 0（相当于吸收所有入射辐射的黑色物体）到 1（相当于反射所有入射辐射的物体）。[2]反射的比例不仅取决于表面本身的属性，还取决于到达地球表面的太阳辐射的光谱和角度分布。[3]这些因素随大气成分、地理位置和时间（见太阳位置）而变化。

- [NASA](https://mynasadata.larc.nasa.gov/mini-lessonactivity/what-albedo) : 反照率是表面反射光的比例。如果全部反射，反照率等于 1；如果 30% 反射，反照率为 0.3。地球表面（大气层、海洋、陆地表面）的反照率决定了有多少进入的太阳能或光会立即被反射回太空。

**关于地表反射率的定义：**

- [Comprehensive Remote Sensing-Earth’s Energy Budget](https://www.sciencedirect.com/topics/earth-and-planetary-sciences/surface-reflectance) : 地表反射率 (ρ)是指在特定入射或观测情况下（定向、锥形和半球形情况），从地球表面反射的入射太阳辐射的分数。不同入射和观测几何图形的反射率量可明确定义为九种情况（Schaepman-Strub 等人，2006 年）：双向反射率、单向反射率、双向反射率和半球反射率、 2006 年）：双向反射率 (BDR)、定向圆锥反射率 (DCR)、定向半球反射率 (DHR)、圆锥定向反射率 (CDR)、双圆锥反射率 (BCR)、圆锥半球反射率 (CHR)、半球定向反射率 (HDR)、半球定向反射率 (HCR) 和双半球反射率 (BHR)。这些反射率量的命名是根据入射和观察几何形状的组合（以入射观察反射率的形式）制定的。
- [GEOGRAPHIC INFORMATION SYSTEMS论坛]([remote sensing - Difference between Albedo and Surface Reflectance - Geographic Information Systems Stack Exchange](https://gis.stackexchange.com/questions/36726/difference-between-albedo-and-surface-reflectance)) : 地表反射率是物体反射太阳辐射的能力。 它被描述为辐射波长的函数，并且仅由物体的物理成分决定。

**albedo 和 reflectance 二者之间的关系：**

- [**Relation of reflectivity to albedo** ](https://gis.stackexchange.com/questions/36726/difference-between-albedo-and-surface-reflectance): 反照率(albedo)是入射**太阳辐射光谱成分**与**地表光谱反射率(reflectance)**的集成结果。在大气层外，太阳辐射的光谱成分相对恒定，在约 0.5µm 处达到峰值，在较短波长处迅速减少到 0.2µm 处的少量，在较长波长处减少得较少，在约 4.0µm 处的少量。大气层会选择性地吸收和散射太阳辐射。因此，在地球表面，太阳辐射的光谱组成因大气条件（如云层、水蒸气和尘埃）和太阳高度的不同而有很大差异（Robinson，1966 年；Dickinson，1983 年）。大多数反照率测量都是在晴空万里、太阳高度较高的条件下进行的（表 A15）。在多云条件下，辐射主要来自可见光谱。这降低了土壤和植被的反照率，但增加了雪的反照率（米勒，1981 年）。当大气浊度较高或太阳在天空中的位置较低时，太阳辐射的光谱分布就会转向光谱的红色和红外线部分。土壤和植被反照率增加，雪反照率降低（Kondrotyev，1973 年）。这种变化表明，要评估地球反照率，就必须同时了解地表的光谱反射率和入射辐射的光谱组成。

- [**Computer Vision: A Modern Approach (2012)** ](https://gis.stackexchange.com/questions/36726/difference-between-albedo-and-surface-reflectance): 随波长变化的漫反射有时也被称为光谱反射率（有时简称为反射率，或不太常见的光谱反照率）。
The wavelength-dependent diffuse albedo is sometimes referred to as the spectral reflectance (sometimes abbreviated to reflectance or, less commonly, spectral albedo).
