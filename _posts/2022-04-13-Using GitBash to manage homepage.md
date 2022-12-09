---
layout:     post
title:      通过Gitbash的方式来管理博客
subtitle:   
date:       2022-04-13
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

1、master是主分支，还可以建一些其他的分支用于开发。
2、git push origin master的意思就是上传本地当前分支代码到master分支。git push是上传本地所有分支代码到远程对应的分支上。


如果出现以下错误，可以通过设置代理端口来解决
fatal: unable to access 'https://github.com/peisipand/peisipand.github.io.git/': Failed to connect to github.com port 443 after 21050 ms: Timed out
```bash
git config --global http.proxy http://127.0.0.1:10809
```
此处的10809是我的端口（可以在设置-网络与Internet-代理-端口中查看）。

遇到报错
$ git push origin master
git@github.com: Permission denied (publickey).
fatal: Could not read from remote repository.

检查
$ git config --global --list
user.email=peisipand@whu.edu.cn
user.name=peisipand
http.sslverify=true
http.proxy=http://127.0.0.1:7890

正常应该是这样
$ ssh -T git@github.com
Hi peisipand! You've successfully authenticated, but GitHub does not provide shell access.


$ git push origin master
fatal: 'origin' does not appear to be a git repository
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.


$ git remote add origin git@github.com:peisipand/peisipand.github.io.git





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
