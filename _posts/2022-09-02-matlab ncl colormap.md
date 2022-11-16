---
layout:     post
title:      matlab获取ncl的colormap
subtitle:   
date:       2022-09-02
author:     Peisipand
header-img: img/art-Anaconda-TensorFlow.jpg
catalog: true
tags:
    - matlab
---


---

[NCL官网](https://blog.csdn.net/peisipand?spm=1001.2101.3001.5343)提供了很多颜色图合集

```bash
function color = ncl_colormap(colorname)
url = ['https://www.ncl.ucar.edu/Document/Graphics/ColorTables/Files/',colorname,'.rgb'];
sourcefile=urlread(url,'get','');
source = strtrim(sourcefile);
source = regexp(source, '\s+', 'split');
source(1:6) = [];
for i=1:(size(source,2))
    color(i) = str2double(source{i});    
end
color = reshape(color,size(color,2) / 3,3);
color = color/255;
end
```


# 参考链接

https://blog.csdn.net/wokaowokaowokao12345/article/details/112969927

https://blog.csdn.net/qq_27984679/article/details/104170418