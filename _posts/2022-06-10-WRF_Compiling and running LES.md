---
layout:     post
title:      WRF LES的编译及使用
subtitle:   LES的编译和运行、位温的设置
date:       2022-06-10
author:     Peisipand
header-img: img/wallhaven-dg3opm.jpg
catalog: true
tags:
    - WRF
---


---



WRF模型有两大模拟类型可以生成初始数据，一个是ideal初始化，一个是real初始化。本文将介绍如何利用WRF-LES在指定风速下模拟气体扩散过程，实现以下效果。

![v3_30m_Q1000](/img/WRF_Compiling and running LES/v3_30m_Q1000.gif)

# 一、理想案例（指定风向、风速等）
## 1.1 编译
为了将tracer添加到WRF的模拟中，必须对以下内容进行编辑：
1. WRFV3/Registry/Registry.EM
[这里](/data/Registry.EM)是我的Registry.EM
2. WRFV3/dyn_em/module_initialize_les.F
[这里](/data/module_initialize_les.F)是我的module_initialize_les.F，这里的目的是初始化羽流（只在开始的时刻排放）。
3. WRFV3/dyn_em/solve_em.F
[这里](/data/solve_em.F)是我的solve_em.F，这里的目的是持续排放。

改完这三个之后需要对wrf重新进行编译,编译步骤如下

`cd /dat01/zppei/WRF`

`./clean -a` （不加-a 清理不干净）

`./configure`  选择GNU dmpar (我选的34)

`./compile em_les >& compile.log`

`cat -n compile.log | grep Error`     检查是否有报错，正常情况下 应该是啥也不显示的

## 1.2 运行

1. 修改input_sounding，这里是设置不同高度的气压、风速等。[这里](/data/input_sounding)是我的input_sounding。

   [混合层高度500](/data/input_sounding_500m)、[混合层高度800](/data/input_sounding_800m)、[混合层高度1100](/data/input_sounding_1100m)

   ![picture1](/img/WRF_Compiling and running LES/1.png)

   这里需要注意的是第2列是位温Potential temperature (K)，并不直接表示气温。这里间接表示了混合层、逆温层的高度。这里的设置会对垂直方向的扩散有很大影响。如果在给定的不同高度上，气温不断随着高度的升高而升高，那么就意味着气体会不断往上飘，然后程序就该报错了，毕竟更高位置处没有指定气象参数。

   **设置位温的依据**

   ```
   % 位温（Potential Temperature）是在绝热条件下，将气块压缩或膨胀到参考压力级别时的温度。它是描述大气中气块垂直运动和对流特性的重要参数。
   % 计算潜在温度的常用方法是使用气象学中的绝热过程假设，即气块在无热交换的条件下进行压缩或膨胀。计算潜在温度需要以下输入数据：
   % 物理温度（Kelvin）：物理温度是指实际的温度观测值。
   % 参考压力级别（通常为1000 hPa或100000 Pa）：这是将气块压缩或膨胀到的目标压力级别。
   % 使用这些输入数据，可以按照以下步骤计算潜在温度：
   % 将物理温度转换为开尔文（Kelvin）单位。
   % 计算位温（Potential Temperature）：
   % θ = T * (P0 / P)^(R/cp)
   % 其中，
   % θ：潜在温度（Potential Temperature）
   % T：物理温度（Kelvin）
   % P0：参考压力级别（Pa）
   % P：气块所在的压力级别（Pa）
   % R：气体常数（通常为287.04 J/(kg·K)）
   % cp：定压比热容（通常为1004 J/(kg·K)）
   
   clc;clear;
   filename = 'adaptor.mars.internal-1684218228.9109106-8058-19-0468b957-036c-4809-b963-6dbffef6462d.nc'; % 来自于https://cds.climate.copernicus.eu/cdsapp#!/yourrequests?tab=form
   ncdisp(filename)
   P = ncread(filename,'level');
   P = double(P);
   P0 = 1000;
   R = 287.04;
   cp = 1004;
   T = ncread(filename,'t');
   [a,b,c,d] = size(T); % lat * lon * levels * times
   T = permute(T(3,3,:,16),[3 2 1]);
   theta = T .* (P0 ./ P) .^ (R / cp);
   
   geo_potential = ncread(filename,'z');
   geo_potential = permute(geo_potential(3,3,:,1),[3 2 1]);
   height = geo_potential2height(geo_potential);
   plot(T,height)
   % %% 混合层高 888m
   % T1 = [290 287.9 285.8 283.6 283.7 284 283.8 283.6 283.35 281.85 280.35 277.05]';
   % height1 = flipud(height(26:37));
   % plot(T1,height1)
   % P1 = flipud(P(26:37));
   % theta1 = T1 .* (P0 ./ P1) .^ (R / cp);
   % [height1 theta1]
   
   %% 500m 混合层高度
   height_500 = [100 300 500 800 1100 1300 1500 6000];
   P_500 = interp1(height-254.36,P,height_500,'spline'); % 根据高度插值压力
   theta_500  = [290 290 290 293  297  299  301 321.4];
   T_500 = theta_500 ./ ((P0 ./ P_500) .^ (R / cp));
   plot(T_500,height_500);
   (T_500(end) - T_500(end-1)) / (height_500(end) - height_500(end-1)) * 1000 %需要大致满足每上升1km，温度降低6度
   
   %% 800m 混合层高度
   height_800 = [100 300 500 800 1100 1300 1500 6000];
   P_800 = interp1(height,P,height_800,'spline'); % 根据高度插值压力
   theta_800  = [290 290 290 290 293  296  299  319.1];
   T_800 = theta_800 ./ ((P0 ./ P_800) .^ (R / cp));
   plot(T_800,height_800);
   (T_800(end) - T_800(end-1)) / (height_800(end) - height_800(end-1)) * 1000 %需要大致满足每上升1km，温度降低6度
   
   %% 1100m 混合层高度
   height_1100 = [100 300 500 800 1100 1300 1500 1700 6000];
   P_1100 = interp1(height,P,height_1100,'spline'); % 根据高度插值压力
   theta_1100  = [290 290 290 290 290  292  295 297 316];
   T_1100 = theta_1100 ./ ((P0 ./ P_1100) .^ (R / cp));
   plot(T_1100,height_1100);
   (T_1100(end) - T_1100(end-1)) / (height_1100(end) - height_1100(end-1)) * 1000 %需要大致满足每上升1km，温度降低6度
   ```

   ```
   function [height] = geo_potential2height(geo_potential)
   %GEO_POTENTIAL2HEIGHT 此处显示有关此函数的摘要
   %   此处显示详细说明
   R_earth = 6400 * 1e3;
   g = 9.8;
   height = (geo_potential .* R_earth) ./ (g .* R_earth - geo_potential);
   end
   ```

   

2. 修改namelist.input，根据实际情况修改一些关键参数。例如

   tke_heat_flux表示地热通量 heat flux。其用的kinetic的单位，k m/s，大约0.1 k m/s ~= 100w/m^2 （[参考链接](https://www.bing.com/search?q=WRF++tke_heat_flux++unit&qs=n&sp=-1&lq=0&pq=wrf+tke_heat_flux++unit&sc=8-23&sk=&cvid=7FBD6CB51EE84ED7AC44C3CB50228741&ghsh=0&ghacc=0&ghpl=&FPIG=2255E39ED8354BA9BC463F8036BA9D0A&first=10&FORM=PORE)）

   [这里](/data/namelist.input)是我的namelist.input

2. 下面就是运行real.exe和wrf.exe了。

​	利用指令 sbatch wrf.sh 在超算中心提交作业。

​	wrf.sh文件如下：
```bash
#!/bin/bash

#SBATCH --account=udhan
#SBATCH --partition=hpib
#SBATCH --nodes=1
#SBATCH --ntasks=16	
#source ~/.bashrc

# 切换目录
cd /dat01/zppei/WRF/test/em_real
#执行wrf.exe程序
mpirun -np 16 ./real.exe
mpirun -np 16 ./wrf.exe
```

# 二、实际案例（使用真实的气象资料）

暂时还没做。

# 三、经验总结

## 3.1 WRF的版本区别

相较于WRFV3，WRFV4有几处不同，比如要在namelist.input最后添加 ideal_case = 9，[数字9代表les](https://github.com/wrf-model/WRF/blob/master/run/README.namelist)

```
 &namelist_quilt
 nio_tasks_per_group = 0,
 nio_groups = 1,
 /
 &ideal
 ideal_case = 9
 /
```

另外，对于WRFV4，/dyn_em/下没有 module_initialize_les.F,9种理想场景被放在了同一个文件里（ module_initialize_ideal.F），[这里](/data/Registry.EM)是我的module_initialize_ideal.F。

## 3.2 改变输出频率

WRF在默认的情况下会每小时输出一次结果，如果想每10分钟，[可以这么做](https://home.chpc.utah.edu/~u0553130/Brian_Blaylock/tracer.html)。

## 3.3 时间步长的设定

根据WRF[官方的指导](https://home.chpc.utah.edu/~u0553130/Brian_Blaylock/tracer.html)，namelist中的时间步长最好设置为6\*dx。 

   time_step: time step for integration in integer seconds (recommended 6\*dx in km for a typical case)

当我们设置的dx = 30m时, time_step = 0.03 * 6 = 0.18，但是time_step 是时间步长的整数部分的。我们需要通过分数的形式（$$
time\_step + \frac{time\_step\_fract\_num}{time\_step\_fract\_den}
$$）来实现。

```
 time_step                           = 0,
 time_step_fract_num                 = 3,
 time_step_fract_den                 = 10,
```

以上的表示时间步长是0.3秒，虽然还没到0.18，但是不会报错了，再进一步的缩短时间步长也可以运行，不过会耽误很长的计算时间。扎不多德勒~

## 3.4 Plume re-enter domain 羽流重新进入区域的问题

一开始只是让程序跑起来，后面需要生成30m分辨率的结果。格网数少了会导致没多大一会儿气体就到了边界，格网数过多会导致模拟速度巨慢，对电脑的配置也是考验。我关心的其实只有排放源附近的，至于吹到很远处的粒子，不在考虑范围。在WRF&MPAS-A forum得到了[解释](https://forum.mmm.ucar.edu/threads/plume-re-enters-the-domain-after-several-hours.11697/#post-30016)。Ps.这里不得不感慨一下美国科研机构慷慨，这么庞大的工程，免费地公开给全世界的人用，使用过程中遇到问题还有耐心的专业人员在线答疑，Respect！

# 参考链接
>1. https://xg1990.com/blog/archives/281
>2. https://home.chpc.utah.edu/~u0553130/Brian_Blaylock/tracer.html