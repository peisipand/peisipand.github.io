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

读论文的时候发现一个好看的图（如下），之前知道这种图用R语言来画非常方便，这次想用MATLAB来实现一下。

很多卫星产品和气象模型喜欢使用nc文件类型，其优点是存储高效。NC文件的结构大致为**变量**和**属性**。属性分为**全局属性**和**变量属性**。

1.创建全局属性

```
copyfile(which("example.nc"),"myfile.nc")
fileattrib("myfile.nc","+w")
creationDate = ncreadatt("myfile.nc","/","creation_date")
```

2.修改变量的属性

```
copyfile(which("example.nc"),"myfile.nc")
fileattrib("myfile.nc","+w")
oldDescription = ncreadatt("myfile.nc","peaks","description")
```
3.以a.nc为样板，创建一个b.nc，并按照修改变量

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

