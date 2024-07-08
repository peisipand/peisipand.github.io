---
layout:     post
title:      使用MATLAB保存NC文件
subtitle:   Save NC file Using MATLAB
date:       2024-05-03
author:     Peisipand
header-img: img/wallhaven-dg3opm.jpg
catalog: true
tags:
    - Matlab
---


---

很多卫星产品和气象模型喜欢使用nc文件类型，其优点是存储高效。NC文件的结构大致为**变量**和**属性**，**属性**又分为**全局属性**和**变量属性**。

# 一、常见函数的基本用法

## 创建NC文件需要三步

### 第一步：创建变量，定义这个变量的维度，以及类型（如double or float）

在名为 `myncclassic.nc` 的文件中创建一个名为 `peaks` 的二维变量。使用 `Dimensions` 名称-值参量指定每个维度的名称和长度。

```
nccreate("myncclassic.nc","emi", "Dimensions",{"lon",360,"lat",180},"Datatype","single");
```

## 第二步：赋予变量具体的数值矩阵

```
ncwrite("myncclassic.nc","emi",ones(360,180))
```

## 第三步：赋予 变量/全局 属性

```
ncwriteatt("myncclassic.nc", "emi", 'units','kg/h'); % 创建emi变量的属性 units
ncwriteatt("myncclassic.nc","/","modification_date",datestr(datetime("now"))) % 创建全局属性 modification_date
```

显示 netCDF 文件的内容。

```
ncdisp("myncclassic.nc")

Format:
           netcdf4_classic
Global Attributes:
           modification_date = '08-Jul-2024 21:15:52'
Dimensions:
           lon = 360
           lat = 180
Variables:
    emi
           Size:       360x180
           Dimensions: lon,lat
           Datatype:   single
           Attributes:
                       units = 'kg/h'
```

除此之外，还有一些常见的函数

### 查看全局属性

```
% matlab自带了example.mc
creationDate = ncreadatt("myfile.nc","/","creation_date")
```

显示属性

```
disp(creationDate)

creationDate =

    '29-Mar-2010'
```

### 修改变量属性

创建 `example.nc` 文件的可写本地副本，并检查 `peaks` 变量的 `description` 属性的值。

```
copyfile(which("example.nc"),"myfile.nc")
fileattrib("myfile.nc","+w")
oldDescription = ncreadatt("myfile.nc","peaks","description")
oldDescription = 
'z = peaks(50);'
```

更新属性并验证其新值。

```
ncwriteatt("myfile.nc","peaks","description","Output of PEAKS")
newDescription = ncreadatt("myfile.nc","peaks","description")
newDescription = 
'Output of PEAKS'
```

目前发现修改变量属性是可以的，不改变变量维度，修改变量也是可以的，但如果想修改变量的维度，就稍微复杂些了。

## 二、具体的应用

以 `AEMISSIONS.2016080100.24.nc` 为样板，创建一个 `modified.nc`，N2O排放改为从EDGAR数据计算的第一个月的排放。

```
%% 第一步裁剪欧洲区域，各行业做累加，得到N2O逐月排放
clc;clear;
file_list_EDGAR_nc = dir(['pzp/pzp/*.nc']);
EU_lat = [31.75,73.75];
EU_lon = [-15.25 34.75];
EU_fluxes_all_sector = zeros(501,421,12);
for i = 1:length(file_list_EDGAR_nc)
    file_name_EDGAR_nc = file_list_EDGAR_nc(i).name;
    file_path_EDGAR_nc = [['pzp/pzp/',file_name_EDGAR_nc]];
    fluxes = ncread(file_path_EDGAR_nc,'fluxes'); % in kg m-2 s-1，空间分辨率是 0.1 度
    lon = ncread(file_path_EDGAR_nc,'lon');
    lat = ncread(file_path_EDGAR_nc,'lat');
    [~,EU_lon_index] = min(abs(lon - EU_lon));
    [~,EU_lat_index] = min(abs(lat - EU_lat));
    EU_fluxes = fluxes(min(EU_lon_index):max(EU_lon_index),min(EU_lat_index):max(EU_lat_index),:);
    EU_fluxes_all_sector = EU_fluxes_all_sector + EU_fluxes;
end
EU_lat = lat(min(EU_lat_index):max(EU_lat_index));
EU_lon = lon(min(EU_lon_index):max(EU_lon_index));
%% 第二步取第一月的数据，单位从 kg m-2 s-1 换成 molec/cm2/s
EU_fluxes_one_month = EU_fluxes_all_sector(:,:,1); % in kg m-2 s-1
% 单位换成 molec/cm2/s
M_N2O = 44.01280; % in g/mol
Na = 6.022 * 1e23; % 阿伏伽德罗常数
EU_fluxes_one_month = EU_fluxes_one_month * 1e3 / M_N2O * Na / 1e4;
imagesc(flipud(EU_fluxes_one_month'))
caxis([0 1e11])
colorbar
title('EDGAR all sector')

%% 第三步 将上一步的结果重采样成 AEMISSIONS 的空间分辨率
% 看看 AEMISSIONS 的信息
AEMISSIONS = 'AEMISSIONS.2016080100.24.nc'; % lat 和 lon 分辨率均为 0.5度
lat_AEMISSIONS = ncread(AEMISSIONS, 'lat'); % 101 * 85 (31.75,73.75)
lon_AEMISSIONS = ncread(AEMISSIONS, 'lon'); % 101 * 85 (-15.25,34.75)
% 高分辨 降采样到 低分辨率
deg = 0.5;
lat_era = min(lat_AEMISSIONS(:)) : deg : max(lat_AEMISSIONS(:));                   
lon_era = min(lon_AEMISSIONS(:)) : deg : max(lon_AEMISSIONS(:)); 

for ii=1:length(lat_era)
	for jj=1:length(lon_era)
			ix_lat = find((lat_era(ii)-deg/2<=EU_lat & EU_lat<=lat_era(ii)+deg/2));
			ix_lon = find((lon_era(jj)-deg/2<=EU_lon & EU_lon<=lon_era(jj)+deg/2));
			if ~isempty(ix_lat) & ~isempty(ix_lon)
				new(jj,ii) = nanmean(nanmean(EU_fluxes_one_month(ix_lon,ix_lat)));
			end
	end
end
imagesc(flipud(new'))
caxis([0 1e11])
colorbar
title('resampled EDGAR all sector')
%% 第四步 将变量 new 输出为 nc
clearvars -except new
N2O = new;
sourceFile = 'AEMISSIONS.2016080100.24.nc'; % 旧文件名，用于参考
%% 时间的维度从 25 变成了 1
info = ncinfo(sourceFile);
info.Dimensions(3).Length = 1;
info.Variables(5).Dimensions(4).Length = 1;
info.Variables(4).Dimensions(2).Length = 1;
newFile = 'modified.nc'; % 新文件名，用于保存修改后的数据
lat = ncread(sourceFile, 'lat');
lon = ncread(sourceFile, 'lon');
species = ncread(sourceFile, 'species');
Times = ncread(sourceFile, 'Times');
Times = Times(:,1);
n_variable = length(info.Variables);
%% 第 4.1 步：创建变量，定义这个变量的维度，以及类型（如double or float）
% 检查是否需要创建新文件（如果文件不存在）
if ~isfile(newFile)
    % 创建与原始文件相同的结构的新文件
    for i = 1:n_variable
        my_cell =  cell(length(info.Variables(i).Dimensions),2);
        for j = 1:length(info.Variables(i).Dimensions)
            my_cell{j,1} = info.Variables(i).Dimensions(j).Name;
            my_cell{j,2} = info.Variables(i).Dimensions(j).Length;
        end
        my_cell = reshape(my_cell',1,length(info.Variables(i).Dimensions)*2);
        nccreate(newFile,info.Variables(i).Name,'Dimensions',my_cell,"Datatype",info.Variables(i).Datatype,"Format","classic");
    end
end
%% 第 4.2 步：赋予变量具体的数值矩阵
% 将修改后的数据写入新文件
for i = 1:n_variable
    ncwrite(newFile, info.Variables(i).Name, eval(info.Variables(i).Name));
end
%% 第 4.3 步：赋予变量属性（在你的这个 nc 里，属性只有 units）
for i = 1:n_variable
    if length(info.Variables(i).Attributes) ~= 0
        ncwriteatt(newFile, info.Variables(i).Name, info.Variables(i).Attributes.Name,info.Variables(i).Attributes.Value);
    end
end

% 检查结果
disp('数据修改完成，并已写入新文件。');
```

