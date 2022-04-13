---
layout:     post
title:      通过Gitbash的方式来管理博客
subtitle:   
date:       2021-04-13
author:     Peisipand
header-img: img/art-Anaconda-TensorFlow.jpg
catalog: true
tags:
    - Github
---






---

# 一、使用VsCode来写Markdown语言的博客

Ctrl + Shift + V 可以编辑查看效果

# 二、保存之后使用Gitbash上传

```bash
git init
```

```bash
git add .
```

```bash
git commit -m "添加文件夹"
```

```bash
git push origin master
```

如果出现以下错误，可以通过设置代理端口来解决
fatal: unable to access 'https://github.com/peisipand/peisipand.github.io.git/': Failed to connect to github.com port 443 after 21050 ms: Timed out
```bash
git config --global http.proxy http://127.0.0.1:10809
```
此处的10809是我的端口（可以在设置-网络与Internet-代理-端口中查看）。