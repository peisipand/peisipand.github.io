---
layout:     post
title:      辐射物理量常用术语
subtitle:   Terminologies commonly used for radiation physical quantities
date:       2024-07-12
author:     Peisipand
header-img: img/wallhaven-dg3opm.jpg
catalog: true
tags:
    - Concept
---

先来了解几个概念

## 漫反射

漫反射，是投射在粗糙表面上的光向各个方向反射的现象。当一束平行的入射光线射到粗糙的表面时，表面会把光线向着四面八方反射，所以入射线虽然互相平行，由于各点的法线方向不一致，造成反射光线向不同的方向无规则地反射，这种反射称之为“漫反射”或“漫射”。这种反射的光称为漫射光。

- **光照强度与观察角度没有关系**：从各个角度观看灯光时，它都具有相同明显的强度
- **光照强度跟灯光的入射角有关系**：如果改变光的入射光方向，可以看到模型表面的光照强度发生了变化

## 辐射源

辐射源可以被简单地划分为点源和面源。点 源是一种理想的情况，辐射源的大小可以忽略不 计，近似为一个点，点源可以向四面八方发射能 量；当辐射源的大小不可忽略不计，不能被近似 为一个点时，需要当作面源。整个面源的辐射可 以基于微面元的积分得到，每个微面元往往被近 似为平面。一个无限大平面可将三维空间分割为 两部分，每个部分都是一个半球空间，辐射可以 从微面元出射到半球空间，也可以从半球空间入 射被微面元所接收。

---

当太阳光照射的角度固定时，不同的地表具有不同的辐射特征，直观的来讲，草地和雪地此时的亮度是不同的，同样地，他们的“辐射状况”也不相同。那么我们该如何定义一个物理量，用来描述地表独有的“辐射状况”呢？

## 辐射能量 radiant energy

遥远感知（remote sensing）利用传感器（detector/sensor）来评价地面辐射。数码相机的CMOS元件（一种感光元件）就是一种最常见的传感器。CMOS元件接收到光能后发生光电效应(Photoelectric Effect)，从而产生电信号被记录下来，最终每个 sensor element 得到一个数值(digital number)，该值越大，表示接收的能量越高。

传感器接收到的物理量是 **辐射能量(radiant energy)**，用符号 $Q$ 来表示，单位是 $J$。

- 然而，如果用两个不同的传感器来测量同一块地表，一个曝光1s，一个曝光1min，曝光时间长的感光元件接收的能量更多。这说明辐射能量不仅与地物辐射有关，而且与传感器参数有关。因此其并不能准确反映地物的真实辐射状况。

## 辐射通量 radiant flux

如果用单位时间的辐射能量来描述，就可以避免不同传感器之间积分时间不一致的问题，这时我们有了一个新的辐射度量--**辐射通量(radiant flux)**，用符号 $\Phi$ 来表示，单位是 $J/s$ 。

- 然而，不同传感器的 sensor element 的物理尺寸(physical size)可能并不一样，尺寸大的 sensor element 必然接收的辐射通量更多。因此也不能准确反映地物的真实辐射状况。

## 辐照度 irradiance

用单位面积内的辐射通量来描述，就可以避免不同 sensor element 尺寸不一致的问题，这时我们有了一个新的辐射度量--**辐照度(irradiance)**，用符号 $E$ 来表示，单位是 $W/m^2$ 。

$$
E = \frac{d\Phi}{dA}
$$

注意：这里的面积就是实际被照的面积，而不是投影面积。

关于辐照度，一个常用的定律是`朗伯余弦定律 (Lambert's cosine law)`。

> 在光学中，朗伯的余弦定律说，从理想的漫反射表面或理想的漫射辐射体观察到的辐射强度或发光强度与入射光的方向与表面法线之间的夹角θ的余弦成正比。该定律也称为余弦发射定律或兰伯特发射定律。它以约翰·海因里希·兰伯特（Johann Heinrich Lambert）的名字命名，来自他于1760年出版的《Photometria》。

来看具体例子

当入射光束与接收面垂直时：

![img](\img\Concept_radiative_trasfer_model\Lambert_cosine_1.png)

此时：

$$
E = \frac{\Phi}{A}
$$
当入射光束与接收面存在夹角时：

![img](\img\Concept_radiative_trasfer_model\Lambert_cosine_2.png)

此时：

$$
E = \frac{\Phi}{A'} = \frac{\Phi cos \theta}{A}
$$
即：表面辐照度与入射光和表面法线之间的夹角余弦成正比。

## 辐射强度 Radiant intensity

- 然而，不同传感器的视场角(field of view, FOV)可能并不一样，大视场角对应的地面范围肯定更大，传感器接收的辐照度通常也会更大。因此也不能准确反映地物的真实辐射状况。

于是，我们尝试在单位空间角度中定义一个新的物理量，来避免视场角差异的问题。由于是在三维空间中定义角度，我们引入立体角(solid angle)，用符号 $\Omega$ 来表示，单位是球面度($steradian,sr$)，概念类比于平面弧度在三维空间中的推广（梁顺林 等，2013）。某立体角的大小可以被简单地定义为该立体角投影到一个以其顶点为球心的球面时，投影的球面面积 $A$ 与球半径 $r$ 平方的比值

$$
\Omega = \frac{A}{r^2}
$$

假设一个球半径为 $r$，其表面积为 $4πr^2$，所以整个球面对球心张成的立体角是 $4π$，而半球空间的立体角为 $2π$。

对于一个以$\theta$为天顶角、$\varphi$ 为方位角的立体角微元 $d\Omega$，它在以其顶点为球心的球面上截得的面积可以近似用矩形的面积求得：

![立体角对应面积额计算](\img\Concept_radiative_trasfer_model\omega.jpeg)

$$
d\Omega = \frac{dA}{r^2}=\frac{rd\theta \cdot r sin\theta d\varphi}{r^2} = sin \theta d\theta d\varphi
$$

**辐射强度（Radiant Intensity）**：点源在某一给定方向上单位立体角内发出的辐射通量， 国际单位制的基本单位为瓦特每球面度 （$W/sr$）。 辐射强度定义式为

$$
I = \frac{d\Phi}{d\Omega}
$$

## 辐射亮度 Radiance

**辐射亮度（Radiance）**：简称辐亮度，描述的是：面辐射源在某一方向，单位立体角，单位时间内， 垂直于辐射方向单位面积上的辐射能量 ，用符号 $L$ 来表示，单位是瓦特每球面度每平方米($W/m^2/sr$)。辐亮度定义式为

$$
L=\frac{\mathrm{d} M}{\mathrm{~d} \Omega \cos \theta}=\frac{\mathrm{d} I}{\mathrm{~d} A \cos \theta}=\frac{\mathrm{d}^2 \Phi}{\mathrm{d} \Omega \mathrm{d} A \cos \theta}
$$

这个时候就会有人疑惑了，**辐亮度是不随观测角度的变化而变化的**。这里为什么表达式中会含有 $cos \theta$。

相比垂直观测时的地表面积 $dA$，当传感器以 $\theta$ 为天顶角倾斜观测时，在同样大小的立体角内，传感器观测到的面积变成了 $dA/cos\theta$。正因为这一特点，对于无限大的 **各向同性** 面辐射源，辐亮度不随观测角度的变化而变化。

一般来说，面辐射源向不同方向的辐射能力是不同的，也就是说，观察者在不同方向上测量的辐亮度值不同。特别地，各角度观察到的辐亮度相同的辐射源称为朗伯体，辐亮度大小和角度有关的辐射源称为非朗伯体。朗伯体的辐亮度跟角度无关。

对于朗伯体，根据辐亮度定义，可以通过积分来计算它在半球空间($2\pi$)内的辐射出射度：

$$
M=\int_{2 \pi} L \cos \theta \mathrm{d} \Omega=L \int_0^{\pi / 2} \cos \theta \sin \theta \mathrm{d} \theta \int_0^{2 \pi} \mathrm{d} \varphi=\pi L
$$

## 总结

通常，辐亮度描述`面光源`（例如地物）很合适，辐射强度描述`点光源`（例如太阳）更为合适。


| 辐射物理量 | 英文              | 常用符号                                | 单位           |
| ---------- | ----------------- | --------------------------------------- | -------------- |
| 辐射能量   | Radiant Energy    | $Q$                                     | $J$            |
| 辐射通量   | Radiant Flux      | $\Phi = \frac{dQ}{dt}$                  | $J/s$ 或者 $W$ |
| 辐照度     | Irradiance        | $E = \frac{d\Phi}{dA}$                  | $W/m^2$        |
| 辐射出射度 | Radiant Exitance  | $M = \frac{d\Phi}{dA}$                  | $W/m^2$        |
| 辐射强度   | Radiant Intensity | $I = \frac{d\Phi}{d\Omega}$             | $W/sr$         |
| 辐射亮度   | Radiance          | $L = \frac{d\Phi}{d\Omega dAcos\theta}$ | $W/m^2/sr$     |

## 参考链接

[辐射物理量概念介绍——阎广建 (遥感学报)](https://www.ygxb.ac.cn/zh/article/doi/10.11834/jrs.20233403/)

[[辐射基础\] 必须要弄懂系列之 (1) 基本辐射度量 - 李二的阳台 - 博客园 (cnblogs.com)](https://www.cnblogs.com/ludwig1860/p/13930745.html)

[Lambert's cosine law](https://en.wikipedia.org/wiki/Lambert's_cosine_law)
