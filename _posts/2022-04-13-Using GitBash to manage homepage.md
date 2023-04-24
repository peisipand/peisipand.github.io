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

# 一、使用编辑器来写Markdown语言的博客

Ctrl + Alt + V 可以查看效果 (VsCode)

Ctrl + / 可以查看效果 (Typora)

# 二、保存之后使用Gitbash上传

```bash
git init (或者 git clone https://github.com/peisipand/peisipand.github.io.git)
```

```bash
git add .
```

```bash
git commit -m "添加文件夹"
```

```bash
git push origin main
```

1、main是主分支，还可以建一些其他的分支用于开发。
2、git push origin main的意思就是上传本地当前分支代码到main分支。git push是上传本地所有分支代码到远程对应的分支上。

# 三、遇到过的报错

>错误1

```
fatal: unable to access 'https://github.com/peisipand/peisipand.github.io.git/': Failed to connect to github.com port 443 after 21050 ms: Timed out
```
可以通过设置代理端口来解决
```bash
git config --global http.proxy http://127.0.0.1:10809
```
此处的10809是我的端口（可以在设置-网络与Internet-代理-端口中查看）。

>错误2
```
$ git push origin master
git@github.com: Permission denied (publickey).
fatal: Could not read from remote repository.
```
检查
```
$ git config --global --list
user.email=peisipand@whu.edu.cn
user.name=peisipand
http.sslverify=true
http.proxy=http://127.0.0.1:7890
```
正常应该是这样
```
$ ssh -T git@github.com
Hi peisipand! You've successfully authenticated, but GitHub does not provide shell access.
```


>错误3(出现在换了电脑上传文件时)

GitHub->Settings->developer settings->Personal access tokens->Tokens->generate new token
```
$ git push origin main
Username for 'https://github.com': peisipand@whu.edu.cn
Password for 'https://peisipand@whu.edu.cn@github.com': 
枚举对象中: 8, 完成.
对象计数中: 100% (8/8), 完成.
使用 12 个线程进行压缩
压缩对象中: 100% (5/5), 完成.
写入对象中: 100% (5/5), 1.73 KiB | 1.73 MiB/s, 完成.
总共 5（差异 3），复用 0（差异 0），包复用 0
remote: Resolving deltas: 100% (3/3), completed with 3 local objects.
To https://github.com/peisipand/peisipand.github.io.git
   cc1fe06..45c9e0e  main -> main
```
