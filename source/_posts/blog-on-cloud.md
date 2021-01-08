---
title: 用GitHub Action部署一个Workers站点
date: 2020-12-23 21:30:02
tags:
- cloudflare
- action
- hexo
categories:
- blog
desc: 使用GitHub Action持续集成部署在Cloudflare Workers上的Hexo博客 Mexii's Blog
---

好像有了一台GitHub的虚拟机?

好像有了一台速度飞起的服务器?

<!--more-->

### 缘起

想要一个访问速度飞快的博客,在看到苏卡卡巨佬的一篇博客[将 Hexo 部署到 Cloudflare Workers Site 上的趟坑记录](https://blog.skk.moe/post/deploy-blog-to-cf-workers-site/)之后开始尝试

其实看完那篇博客就基本上可以知道如何部署了,下面只是我的踩坑记录

### 趟坑

整个部署的基本流程是这样的:先在本地完成一系列相关配置,然后配置Action完成原本在本地的部署工作

我之前写的博文很少,加之使用了一个需要配置很久的主题,从零开始配置hexo想必要比将已有的博客搬到云端要来得简单

新建一个目录,完成一系列初始化

```bash
hexo init
git init
```

在GitHub新开一个repo,然后把远程仓库设置过去

```bash
// 这里没命令,我git只会使用vscode的图形界面操作和git push
```

#### submodule问题

对于主题,找一个简单一点的主题,避重就轻嘛

这是我踩的第一个坑,注意告诉git这个主题repo是一个子模块,否则Action build的时候checkout部分会报错

原因我不是特别清楚,出现这个情况说明push到GitHub时,GitHub把themes文件夹下面的主题当作一个submodule,如果没有同时push那个`.gitmodule`文件,在checkout部分就会报错

一开始我是顺着这个原因,创建了内容如下的文件

```.gitmodules
// Google will tell you everything,可以查询相关命令完成
// 也可以自己新建一个`.gitmodules`文件,填入

[submodule "hexo-theme-light"]
	path = themes/light
	url = https://github.com/tommy351/hexo-theme-light
```

记得配置`_config.yml`,然后`hexo g`,`hexo s`检查一下是否成功生成站点

后来我换了一个差不多已经停止maintain的主题,干脆直接删掉了主题repo下面的.git文件夹,这样push到GitHub时,GitHub不会把它当作一个submodule了

#### 主题配置问题

查阅Hexo的文档可知,默认情况下网站目录下的`_config.yml`文件的`theme_config`节点配置会覆盖主题文件夹下的配置,结果是有些主题也做到了反向覆盖...

最好还是自己fork一份主题吧.

#### 配置wrangler

然后是配置wrangler,具体看skk的博客就可以了

当初我用这个CLI最麻烦的问题,是不知道为什么安装时下载极慢,但官方支持manual安装,下载可执行文件然后扔到某个$PATH路径下就可以开始使用了

cloudflare的命令行工具一堆emoji,总有点出戏

要注意的一点是,如果你要使用自己的域名,

1. 需要将一个域名交由cloudflare管理(设置`Nameserver`)

2. 在域名管理页面的DNS配置页面,给要使用的域名添加记录(可以指向任意地址,推荐添加CNAME记录指向Workers的dev子域地址),代理状态选择`已代理`

3. 在域名管理页面找到`zone id`,填入配置文件`wrangler.toml`,并配置`route:[third-level-domain]/*`(自行替换相关字段)

如果不需要的话就是直接使用`wrangler.toml`下的`name`参数和你的dev子域形成网址;登陆cloudflare可以查看到网址

配置好后,试着预览一次,预览成功就可以开始配置GitHub Action了

#### 配置Action

Action的配置文件是`.github/workflows/`下的任意`yaml`文件,每个`yaml`文件都会在符合条件的时候执行

查阅GitHub的文档或者看现成的配置文件都可以看懂Action是怎样照这配置文件执行的,需要注意的是,checkout环节会把repo clone到虚拟机工作目录下,如果有多个repo,就要进行多次checkout

在配置供Action将网站部署到Workers的secrets时,要注意,GitHub现在提供了两类secrets,要在setting中使用repo secrets

在本地写完配置文件,检查无误后,push到GitHub即可

### Summary

我的这个CI部署基本上是完全模拟了本地的的情况,就是Action的虚拟机会拉取我在GitHub上的repo(repo里只有基本的东西,而且按照一开始我们`hexo init`生成的`.gitignore`规定的,repo里不会有`public`和`node_modules`文件夹,这些文件可以在虚拟机里生成了),然后在虚拟机里编译,生成站点以及部署到Workers.

这个过程可以让你知道,我们处理的文件中最重要的,是我们写的Markdown文章和配置文件,我们把这些东西copy似地放到虚拟机上,剩下的交给“自动化”解决

当然这只是CI的初衷之一.

[利用 GitHub Actions 自动部署 Hexo 博客](https://sanonz.github.io/2020/deploy-a-hexo-blog-from-github-actions/)

[GitHub Action Docs](https://docs.github.com/cn/free-pro-team@latest/actions)

[Cloudflare Workers Docs](https://workers.cloudflare.com/docs)