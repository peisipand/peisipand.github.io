---
layout:     post
title:      个人主页搭建详细教程-基于github仓库和Jekyll
subtitle:   
date:       2022-04-12
author:     Peisipand
header-img: img/wallhaven-dg3opm.jpg
catalog: true
tags:
    - Web
---

拥有自己的主页是一件很酷的事，可以在里面记录自己的学习过程，记笔记，分享经验，分享生活...

之前一直在用CSDN写博客[peisipand](https://blog.csdn.net/peisipand?spm=1001.2101.3001.5343)，有些内容很容易被判定为敏感内容，界面也不是很好看。直到有一天看到一位博主分享的GitHub搭建主页的博客...

下面将以我搭建博客为例，详细介绍每一步的流程，希望可以帮到一些人。
# 一、GitHub准备工作
首先需要注册自己的GitHub账号，这个步骤相关教程较多，不具体介绍，只要掌握了科学上网的姿势即可，之后会考虑介绍如何搭建梯子（在Digital Ocean上，首年仅需5美元，再也不用担心vpn跑路问题）。

## 1.1、创建仓库
首先New一个Repository
![picture1](/img/Web_build_my_blog/1.jpg)

这里的需要根据自己的 **username** 替换掉我这里的 **peisipand**
![picture1](/img/Web_build_my_blog/2.jpg)

## 1.2 下载模板
github官方也提供了一些模板，但是颜值不是特别在线，我这里使用的是一位大佬[Hux](https://github.com/Huxpro/huxpro.github.io)上传的一套模板。由于其修改起来非常方便，颜值也非常在线，被很多人当做个人博客[WangPei](https://wangpei.ink/), [qiubaiying](https://qiubaiying.github.io/)的模板。在此表示感谢！

比较简单的下载方式：Code-Download Zip
![picture1](/img/Web_build_my_blog/3.jpg)

# 二、在本地修改模板
## 2.1最关键信息修改
下载到本地之后解压，先对最关键的部分(_config.yml)进行修改，这里的 **peisipand** 均需替换成自己的username。（这里我用的是Vscode）。
![picture1](/img/Web_build_my_blog/4.jpg)


## 2.2 Push代码到github
下面需要把这些代码上传到自己的仓库。这里可以利用git bash的方式来上传,git是一个管理代码十分方便的工具。
1. 首先去Git的[官网](https://git-scm.com/)安装，装完只有在文件管理器右键就可以显示 -Git Bash Here；
2. 然后在解压后的那个文件夹 右键-Git Bash Here；
3. 具体如何上传和常见问题解决可以参考[上篇](https://peisipand.top/2021/04/13/GitBash/)

# 三、进阶教程
## 3.1 照片和文字的替换
img文件夹中放入自己的照片，主页侧栏的个人照在_config.yml中修改照片路径
![picture1](/img/Web_build_my_blog/5.jpg)
**ABOUT**中的文字个人介绍在*about.html*(需要用Vscode或者记事本打开)中修改, 首页的文字在*index.html*中修改。

## 3.2 写博客
博客以markdown的语法来写，并放在_posts文件夹下面。可以参考作者提供的博客模板
## 3.1 开启gittalk
gittalk为博客下面的评论功能，相当于就是把github中issues的评论搬到这里。
第一步：[申请密钥](https://github.com/settings/applications/new)
![picture1](/img/Web_build_my_blog/6.jpg)
这里的Client ID和client secret之后要用到
![picture1](/img/Web_build_my_blog/7.jpg)
第二步：在_config.yml中修改
同样的，这里的**peisipand** 均需替换成自己的username。Client ID和client secret换成自己的
![picture1](/img/Web_build_my_blog/8.jpg)

## 3.2 域名修改
目前我的域名还是 peisipand.github.io ，下面介绍如何申请新的域名来指向我的博客。 Setting-Pages-Custom domain 中填入自己申请到的新域名，Save。
![picture1](/img/Web_build_my_blog/9.jpg)

### 3.2.1 免费域名申请
[freenom](https://my.freenom.com/clientarea.php?action=domains)上可以免费申请域名(.ml 后缀)使用一年，这里我参考了[Leftpocket](https://left-pocket.github.io/post/hugo/hugo_dns/)的博客中 申请域名、DNS解析 部分。

完成这一步之后需要在github上告诉它你有了新域名。

### 3.2.2 腾讯域名购买(.top后缀)
.com .cn这种比较狠的域名肯定就不要想了，买个.top倒还可行，完成个人认证后，.top后缀的域名首年只需**1元**。
购买过程比较简单，这里就不再描述。[域名注册传送门](https://dnspod.cloud.tencent.com/?from=qcloudHpProductDns/)。
域名买完之后，[DNS的解析](https://console.cloud.tencent.com/cns)和github域名绑定与上节相同。

## 3.3 访问统计
这个我自己还没摸清楚，申请了Google Analyze的code进行了替换，但是主页还是没效果，之后如果搞定的话会接着分享。

---
**Note：** 每次在本地修改完上传github后，github会对这些代码进行编译，如果编译成功（也就是说上传完之后需要等个一分钟左右，主页才会随之更新），才可以通过域名访问到自己的博客。Actions中可以查看编辑是否成功。
![picture1](/img/Web_build_my_blog/10.jpg)

# 四、使用编辑器来写Markdown语言的博客

Ctrl + Alt + V 可以查看效果 (VsCode)

Ctrl + / 可以查看效果 (Typora)

# 五、使用Gitbash上传

```bash
git init (或者 git clone https://github.com/peisipand/peisipand.github.io.git)
```

```bash
git add .
```

```bash
git commit -m "1st commit"
```

```bash
git push origin main
```

1、main是主分支，还可以建一些其他的分支用于开发。
2、git push origin main的意思就是上传本地当前分支代码到main分支。git push是上传本地所有分支代码到远程对应的分支上。

# 六、遇到过的报错

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