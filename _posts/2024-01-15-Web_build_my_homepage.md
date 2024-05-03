---
layout:     post
title:      个人学术主页搭建详细教程-基于HugoBlox搭建
subtitle:   
date:       2024-01-15
author:     Peisipand
header-img: img/wallhaven-dg3opm.jpg
catalog: true
tags:
    - Web
---

# 一、概念和搭建思路

## 1.1 GitHub Pages、 Hugo 是什么

* GitHub Pages 是一组静态网页集合（Static Web Page），这些静态网页由 GitHub 托管（host）和发布，所以是 GitHub + Pages。
* Hugo 是用 Go 语言写的静态网站生成器（Static Site Generator）。可以把 Markdown 文件转化成 HTML 文件。

## 1.2 网站搭建思路

1. 创建 1 个 GitHub 仓库,用来存放 Markdown 源文件

2. Hugo 根据 Markdown 源文件生成的静态 HTML 文件，然后部署到远端 GitHub Pages 仓库中。

# 二、Github 仓库创建

## 2.1 创建一个 Github 账号

## 2.2 创建 GitHub Pages 仓库

这里我用的是 [HugoBlox](https://hugoblox.com/templates/details/academic-cv/)提供的 Academic-CV 模板。点击 Edit 进入仓库创建界面。
![Alt text](/img/Web_build_my_homepage/hugo_template.png)

---

![Alt text](/img/Web_build_my_homepage/github_new_repo.png)

1. 命名 GitHub Pages 仓库，这个仓库必须使用特殊的命名格式 <username.github.io>， username 是自己的 GitHub 的用户名。
2. 勾选 Public，设置为公开仓库。

# 三、Git和VS Code安装

在本地的电脑上，还需要安装两个软件，一个是[Git](https://git-scm.com/)，一个是[VS Code](https://code.visualstudio.com/)。Git 是一个版本控制系统，是我们连接 Github 的钥匙；VS Code 是微软出品的一款程序员使用的开发集成工具，里面提供了非常多的插件，用来写 MD 文件非常方便。

## 3.1 Git的安装与配置

Git安装完之后，需要在本地电脑生成 SSH Key 密钥，然后将其添加到 GitHub，就实现了本地电脑和 GitHub 服务器安全连接，可以把本地仓库推送到 GitHub 远程仓库，或把 GitHub 远程仓库拉取到本地仓库，即两台电脑间的数据交换。

>SSH(Secure Shell) 是允许两台电脑之间通过安全连接进行数据交换的网络协议。



### 3.1.1 生成SSH Key

验证本地电脑是否有SSH Key

```
ls -al ~/.ssh
```

看下返回的结果中是否已经存在了.pub结尾的文件。

如果有.pub结尾的文件，直接打开并复制：

```
cat ~/.ssh/id_rsa.pub
```

如果没有，则先创建再复制。

```
//输入下面的代码，获取密钥（双引号内容为你自己注册账号时的邮箱）：
$ ssh-keygen -t rsa -C "email@xxx.com"
```

### 3.1.2 添加SSH Key到Github

头像-GitHub Settings-SSH and GPG keys ，点击 New SSH key，并将上一步复制的内容贴进来。

![Alt text](/img/Web_build_my_homepage/github_setting.png)

### 3.1.3 测试 SSH Key 是否添加成功

1. 选择一个要用 SSH Key clone 的仓库，复制这个仓库的 SSH 链接。

![Alt text](/img/Web_build_my_homepage/github_https.png)

2. 在终端输入：

```
git clone [ssh-url]
```

3. clone 成功即代表 SSH Key 添加成功。

![Alt text](/img/Web_build_my_homepage/github_clone.png)

## 3.2 VS Code的安装与配置

进入 [VS Code官网](https://code.visualstudio.com/) ，选择自己的版本下载，我这里是 Mac 版本。如果安装不熟悉，就一路默认配置。安装完成，打开 VS Code 软件。左下角-设置-Extensions，安装所需插件。在这里推荐几个必备和好用的插件。

* ``Markdown All in One`` 使用 Markdown 写作的利器。必备
* ``Chinese Language Pack for Visual Studio Code`` VS Code 的汉化插件。 推荐
* ``Material Icon Theme`` 一套个性的图标主题。 推荐
* ``Paste Image`` 需要插入图片时，可以直接粘贴。 推荐
* ``Markdown Preview Enhanced`` 拥有飘逸的 Markdown 写作体验。 推荐

# 四、Hugo 创建网站

## 4.1 在线同步到本地

1. 完成上述操作后，我们可以通过``git clone``命令将创建的仓库文件夹克隆至本地电脑。（该步骤不推荐用下载的方式，否则下载后的文件夹和该仓库没有建立连接，后续 本地上传仓库时麻烦一些。）

```
git clone https://github.com/zppei97/HugoFiles.git
cd HugoFiles 
hugo new site zppei-website
```

2. 按照 [HugoBlox](https://docs.hugoblox.com/tutorial/resume/) 提供的教程一步步来修改模板即可。其中有几步非常关键的操作：

* GitHub-Setting-Action-Allow GitHub Actions to create and approve pull requests(勾选Allow)，这一步和教程里稍微不同，可能是由于 Github 网址更新的原因。
  ![Alt text](/img/Web_build_my_homepage/github_setting_pull_requests.png)
* 上传完 ``publications.bib``后，打开 仓库-Pull requests-查看修改并 Merge 。这一步的目的是为了根据这个 bib 文件，模板的程序自动生成文献对应的文件夹。这一步完成之后，我们会发现``content``文件夹下多了个 ``publication``文件夹，但本地的电脑还未生成，所以需要跟一步 git 命令，使得本地的文件与在线的同步。

```
git pull
```

## 4.2 本地同步到在线

在本地修改完模板里的内容后，利用 push 命令将本地的文件夹推至在线仓库。

```
git add . 
git commit -m "commit message"
git push origin main
```

当遇到特殊情况，比如在线仓库的文件比本地的多时，可能会push 不上去，这时就需要暴力 push，命令如下：

```
git push -f origin main
```

# 五、建议和遇到的问题

## 5.1 边修改边 push

刚开始修改模板时，建议修改一点，就 push 一点。这样就算网页编译失败了，我们可以随时回溯到任意一个版本，非常的方便。

## 5.2 Galley 图片上传的问题

当我尝试把自己的图片上传至 ``/assets/media/albums/demo``时，发现网站就会编译失败，查看错误日志也看不出什么信息。我怀疑过图片格式（jpg/png），图片大小（是不是超过某尺寸了），文件命名格式，但一一排查都没解决问题。甚至出现了一个很迷惑的现象：编译成功 -> 添加自己的图片至文件夹 -> push -> 编译失败 -> 本地文件夹删除刚添加的图片 -> push -> 此时还是会失败。
具体报错如下：

```
"/home/runner/work/zppei97.github.io/zppei97.github.io/content/_index.md:1:1": failed to render shortcode "gallery": failed to process shortcode: "/tmp/hugo_cache_runner/modules/filecache/modules/pkg/mod/github.com/!hugo!blox/hugo-blox-builder/modules/blox-bootstrap/v5@v5.9.6/layouts/shortcodes/gallery.html:34:17": execute of template failed: template: shortcodes/gallery.html:34:17: executing "shortcodes/gallery.html" at <.Fit>: error calling Fit: this method is only available for image resources
```

该错误信息可以在 仓库-Actions 中查看。该问题卡了我很久，开始不知道什么原因的时候，发现只有 版本回溯可以解决该问题，所以这就提现了 及时 push 的重要性。版本回溯的命令如下：

```
git log --pretty=oneline
git reset --hard <id> #此处的 id 应为上一句中查询的 id
git add .
git commit -m "back commit"
git push -f origin main
```

后面在一个[博客](https://discourse.gohugo.io/t/gallery-module-error-message/46307/1)上找到了解释，给出的原因是上传新的图片会随机的导致 在线仓库/assets/media/albums/demo/中出现不该出现的``.Ds_Store``文件，我们可以直接在网页上删除该文件，这时就会发现 Actions 中会自动重新编译，就可以编译成功了。