---
layout:     post
title:      辐射传输模型
subtitle:   Radiative transfer
date:       2024-07-08
author:     Peisipand
header-img: img/wallhaven-dg3opm.jpg
catalog: true
tags:
    - Concept
---

> 表观反射率: apparent reflectance
>
> 地表反射率: surface reflectance

The apparent reflectance $\rho^*\left(\theta_s, \varphi_s, \theta_v, \varphi_v, \lambda\right)$ is used in the formation of the radiative transfer problem. The definition of apparent reflectance is

$$
\rho^*\left(\theta_s, \varphi_s, \theta_v, \varphi_v, \lambda\right)=\frac{\pi L\left(\theta_s, \varphi_s, \theta_v, \varphi_v, \lambda\right)}{\mu_s E_s(\lambda)},(1)
$$

where $\theta_s$ is the solar zenith angle, $\varphi_s$ is the solar azimuth angle, $\theta_v$ is the sensor zenith angle,  $\varphi_v$ is the sensor azimuth angle, $\lambda$ is wavelength, $L$ is the radiance measured at the satellite, $E_s$ is the solar flux at the top of the atmosphere (TOA), and $\mu_s = cos \theta_s$. According to Tanre et al. (1986), for a horizontal surface of uniform Lambertian reflectance, $\rho^*\left(\theta_s, \varphi_s, \theta_v, \varphi_v, \lambda\right)$ can be expressed as

$$
\begin{gathered}\rho^*\left(\theta_s, \varphi_s, \theta_v, \varphi_v, \lambda\right)=T_g\left(\theta_s, \theta_v, \lambda\right) \\ \times\left[\rho_a\left(\theta_s, \varphi_s, \theta_v, \varphi_v, \lambda\right)+\frac{T\left(\theta_s, \lambda\right) T\left(\theta_v, \lambda\right) \rho(\lambda)}{1-\rho(\lambda) S(\lambda)}\right]\end{gathered},(2)
$$

where $T_g$ is the total gaseous transmittance in the Sun-surface-sensor path, $\rho_a$ is the atmospheric reflectance, which is related to the path radiance resulted from atmospheric scattering, $T\left(\theta_s\right)$ is the downward scattering transmittance, $T\left(\theta_v\right)$  is the upward scattering transmittance, $S$ is the spherical albedo of the atmosphere, and $\rho$ is the surface refectance. $T_g$ is expressed as

$$
T_g\left(\theta_s, \theta_v, \lambda\right)=\prod_{i=1}^n T_{g i}\left(\theta_s, \theta_i, \lambda\right),(3)
$$

where $T_{gi}$  is the transmittance of the $i$th gas in the Sun-surface-sensor path and $n$ is the number of gases. $T_g$ is calculated by assuming that there is no atmospheric scattering. The transmittances in Eq. (3) refer to the average transmittances over narrow spectral intervals (a few nanometers).

The scattering terms, $\rho$, $T(\theta_s)$, $T(\theta_s)$, and $S$, are calculated by assuming that there are no atmospheric gaseous absorptions. In the real atmosphere, the scattering and absorption processes occur simultaneously. Equation (2) treats these as two independent processes. The coupling effects between the two processes are neglected. The coupling effects are small in regions where the atmospheric gaseous absorptions are weak and in regions where the scattering effects are small. Equation (3) is strictly correct only in regions where there are no overlapping absorptions by different gases. It remains a good approximation for the 0.4-2.5 $\mu$m region because little overlapping absorptions occur in this region. 

The 5S code was originally designed to simulate radiances measured from satellite platforms using geometric and atmospheric models and using surface reflectances as boundary conditions. However, the code can be modified for retrieving surface reflectances from measured radiances (Teillet, 1989). Solving Eq. (2) for $\rho$ yields

$$
\begin{aligned} \rho(\lambda)= & \left(\frac{\rho^*\left(\theta_s, \varphi_s, \theta_v, \lambda\right)}{T_g\left(\theta_s, \theta_v, \lambda\right)}-\rho_a\left(\theta_s, \varphi_s, \theta_v, \varphi_v, \lambda\right)\right) \\ & \times\left[ T\left(\theta_s, \lambda\right) T\left(\theta_v, \lambda\right) + S(\lambda)\left(\frac{\rho^*\left(\theta_s, \varphi_s, \theta_v, \varphi_v, \lambda\right)}{T_g\left(\theta_s, \theta_v, \lambda\right)} -\rho_a\left(\theta_s, \varphi_s, \theta_v, \varphi_v, \lambda\right)\right) \right]^{-1} \end{aligned}
$$

Given a satellite measured radiance, the surface reflectance can be derived according to Eqs. (1) and (4).



---


Reference:

Derivation of Scaled Surface Reflectances from AVIRIS Data, Bo-Cai Gao et al, 1993, RSE.  