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

## 一、常见函数的基本用法

### 创建NC文件

创建一个名为 `myexample.nc` 的 netCDF 文件，其中包含名为 `Var1` 的变量。

```
nccreate("myexample.nc","Var1")
```

在同一文件中创建另一个变量。

```
nccreate("myexample.nc","Var2")
```

显示 netCDF 文件的内容。

```
ncdisp("myexample.nc")
Source:
           pwd\myexample.nc
Format:
           netcdf4_classic
Variables:
    Var1
           Size:       1x1
           Dimensions: 
           Datatype:   double
    Var2
           Size:       1x1
           Dimensions: 
           Datatype:   double
```

### 指定变量维度

在名为 `myncclassic.nc` 的文件中创建一个名为 `peaks` 的二维变量。使用 `Dimensions` 名称-值参量指定每个维度的名称和长度。

```
nccreate("myncclassic.nc","peaks", "Dimensions",{"r",300,"c",400})
```

向变量中写入数据。

```
ncwrite("myncclassic.nc","peaks",peaks(100))
```

显示 netCDF 文件的内容。

```
ncdisp("myncclassic.nc")
Source:
           F:\他人\史天奇\myncclassic.nc
Format:
           netcdf4_classic
Dimensions:
           r = 300
           c = 400
Variables:
    peaks
           Size:       300x400
           Dimensions: r,c
           Datatype:   double
```

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

### 创建全局属性

创建 `example.nc` 文件的可写本地副本，并检查其全局 `creation_date` 属性的值。

```
copyfile(which("example.nc"),"myfile.nc")
fileattrib("myfile.nc","+w") % 文件可读写
ncwriteatt("myfile.nc","/","modification_date",datestr(datetime("now"))) % 创建属性 modification_date
modificationDate = ncreadatt("myfile.nc","/","modification_date") % 读取创建的属性
modificationDate =
    '08-Jul-2024 12:48:15'
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

## 二、具体的应用

以 `AEMISSIONS.2005010100.24.nc` 为样板，创建一个 `modified.nc`，并根据要求，修改变量。

```
clc;clear;
% 要读取的原始文件名
sourceFile = 'AEMISSIONS.2005010100.24.nc'; % 旧文件名，用于参考
info = ncinfo(sourceFile);
newFile = 'modified.nc'; % 新文件名，用于保存修改后的数据

% 读取变量 'N2O'
N2O = ncread(sourceFile, 'N2O');
% 修改数据，这里是一个示例，比如乘以2
N2O = N2O .* 2;
lat = ncread(sourceFile, 'lat');
lon = ncread(sourceFile, 'lon');
species = ncread(sourceFile, 'species');
Times = ncread(sourceFile, 'Times');

n_variable = length(info.Variables);
%% 第一步：创建变量，定义这个变量的维度，以及类型（如double or float）
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
%% 第二步：赋予变量具体的数值矩阵
% 将修改后的数据写入新文件
for i = 1:n_variable
    ncwrite(newFile, info.Variables(i).Name, eval(info.Variables(i).Name));
end
%% 第三步：赋予变量属性（在你的这个 nc 里，属性只有 units）
for i = 1:n_variable
    if length(info.Variables(i).Attributes) ~= 0
        ncwriteatt(newFile, info.Variables(i).Name, info.Variables(i).Attributes.Name,info.Variables(i).Attributes.Value);
    end
end

% 检查结果
disp('数据修改完成，并已写入新文件。');
```

