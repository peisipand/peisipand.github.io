---
layout:     post
title:      在PowerPoint里插入LaTeX公式
subtitle:   Insert LATEX equation in PowerPoint
date:       2024-07-03
author:     Peisipand
header-img: img/wallhaven-dg3opm.jpg
catalog: true
tags:
    - Software
---


---

本文详细了如何使用 [IguanaTex插件](https://www.jonathanleroux.org/software/iguanatex/) 实现在PowerPoint 中优雅的输入公式，输出的格式支持**矢量**和**栅格**。

### 为什么要在ppt里装这个插件，原装的不好么？

如果公式少，用不到多个文档之间的复制粘贴，ppt自带的公式功能就够用了，ppt自带的公式支持点击选取，也支持Latex代码转换，插入-公式-[输入 a_x^2]-设计-专业型，即可完成Latex代码与公式的转换，对应的，再点击 线性 也可以重新回到 Latex代码。

![image-20240705155214129](\img\Software_PPT Latex\1.png)

但有时转换会不太方便，比如手动选择的特殊符号无法转成 Latex代码，一直点来点去还是不够优雅。

### 为什么不用MathType插件？

MathType插件也是很好用的，公式支持手动选择和Latex代码转换，不过想免费使用的话还得破解，隔断时间就不让用了。

### IguanaTex的优势是什么

<u>免费，安装方便，操作简单，易于公式代码的复制粘贴。</u>

进入主题，开始介绍如何Windows版本的安装，其他版本的请参考 [IguanaTex官网](https://www.jonathanleroux.org/software/iguanatex/download.html)。

1. 下载 [IguanaTex v1.58 (.ppam)](https://www.jonathanleroux.org/software/iguanatex/IguanaTex_v1_58.ppam) (October 10, 2020)

2. PPT-文件-选项-加载项-管理[切换到PowerPoint加载项]-转到-添加下载好的 ppam

3. lguanaTeX 选项卡-Main Settings

   ![image-20240705161222080](\img\Software_PPT Latex\2.png)

   如果只需要输出bitmap格式（可以理解为就是栅格，如果手动地拉伸图片，字母边缘会不清晰；如果是撒矢量的话，随便拉伸都是清晰的），下面的 ghostscript, ImageMagick这些就不需要下载了。

其实到这一步就已经够用了，如果想让输出的公式大一些，在 New LaTeX display/Edit LaTeX display 中修改 size 就可以了，这样就可以保证输出的所有公式大小统一，手动拉伸就很难把握了。

如果非想要输出 vector 格式，那就把下面这些下载了。

- (Optional): [GhostScript](https://ghostscript.com/releases/gsdnld.html) and [ImageMagick](https://www.imagemagick.org/script/download.php#windows)
- (Optional): [TeX2img](https://github.com/abenori/TeX2img)

注意：**ghostscript的版本只能是9.18**，别的好几个版本试了都不行。



参考链接：

[Download IguanaTex (jonathanleroux.org)](https://www.jonathanleroux.org/software/iguanatex/download.html)

[在PowerPoint里优雅的插入LaTeX公式 (lbqaq.top)](https://www.lbqaq.top/p/latex-in-ppt/)

[PPT如何安装使用lguanaTex插件并输出矢量图-CSDN博客](https://blog.csdn.net/HandsomeEric/article/details/139795201)
