---
title: 用GitHub Action部署一个Workers站点
date: 2020-12-23 13:30:02
updated: 2021-03-21 14:25:00
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

> 

### 缘起

想要一个访问速度飞快的博客,在看到苏卡卡巨佬的一篇博客[将 Hexo 部署到 Cloudflare Workers Site 上的趟坑记录](https://blog.skk.moe/post/deploy-blog-to-cf-workers-site/)之后开始尝试

其实看完那篇博客就基本上可以知道如何部署了,下面只是我的踩坑记录

### 趟坑

整个部署的基本流程是这样的:先在本地完成一系列相关配置,然后配置Action完成原本在本地的部署工作

我之前写的博文很少,加之使用了一个需要配置很久的主题,从零开始配置hexo想必要比将已有的博客搬到云端要来得简单

首先完成一系列初始化

+ 安装git, nodejs
+ 使用npm或者yarn安装hexo-cli
+ 新建并初始化一个目录

	```bash
	mkdir myblog && cd myblog
	hexo init
	git init
	```

在GitHub新开一个repo,然后把远程仓库设置过去

```bash
git remote add <name> <url>
```

#### submodule问题

对于主题,我选择找一个简单一点的,避重就轻方便配置,并且将它设置为一个submodule

```bash
// 格式
// git submodule add <url> <relative-path>
// 我是这样的
git submodule add git@github.com:weremexii/hexo-theme-apollo.git theme/apollo
```

注意在配置GitHub Action的时候,`actions/checkout@v2`需要配置`submodules: true`参数后才能拉取主题源码

#### 主题配置问题

查阅Hexo的文档可知,默认情况下网站目录下的`_config.yml`文件的`theme_config`节点配置或者网站目录下的`_config.<theme_name>.yml`配置会覆盖主题文件夹下的配置,结果是有些主题却做到了反向覆盖...

选择主题的时候要阅读配置文档

#### 配置本地wrangler

然后是配置wrangler,具体看skk的博客就可以了

<s>当初我用这个CLI最麻烦的问题,是不知道为什么安装时下载极慢,但官方支持manual安装,下载可执行文件然后扔到某个$PATH路径下就可以开始使用了</s>

这种问题要怪自己的网络环境和包管理配置

<s>cloudflare的命令行工具一堆emoji,总有点出戏</s>

要注意的一点是,如果你要使用自己的域名,

1. 需要将一个域名交由cloudflare管理(设置`Nameserver`)

2. 在域名管理页面的DNS配置页面,给要使用的域名添加记录(可以指向任意地址,推荐添加CNAME记录指向Workers的dev子域地址),代理状态选择`已代理`

3. 在域名管理页面找到`zone id`,填入配置文件`wrangler.toml`,并配置`route:[third-level-domain]/*`(自行替换相关字段)

如果不需要的话就是直接使用`wrangler.toml`下的`name`参数和你的dev子域形成网址;登陆cloudflare可以查看到网址

配置好后,试着预览一次,预览成功就可以开始配置GitHub Action了

#### 配置Action

Action的配置文件是`.github/workflows/`下的任意`yaml`文件,每个`yaml`文件都会在符合条件的时候执行

查阅GitHub的文档或者看现成的配置文件(比如我的或者skk的)都可以看懂Action是怎样照这配置文件执行的,需要注意的是,checkout环节会把repo clone到虚拟机工作目录下

在配置`cloudflare/wrangler-action`的`API_Token`时,要注意,GitHub现在提供了两类secrets,要在setting中使用repo secrets

在本地写完配置文件,检查无误后,push到GitHub即可

#### 无密化

在上面的配置过程中我们其实把部署到workers需要的`account_id`和`zone_id`明文写在了文件里,这样多多少少有泄露的风险,好在`cloudflare/wrangler-action`工作的时候支持从环境变量中读取这两个参数

在Action配置`on`字段之后,添加env字段.注意双大括号内的变量名称要和你设置的secrets一致

```yml
env:
  CF_ACCOUNT_ID: ${{ secrets.CF_WORKERS_ACCOUNT_ID }}
  CF_ZONE_ID: ${{ secrets.CF_WORKERS_ZONE_ID }}
```

删除`wrangler.toml`中相关字段后测试一下

### Summary

我的这个CI部署基本上是完全模拟了本地的的情况,就是Action的虚拟机会拉取我在GitHub上的repo(repo里只有基本的东西,而且按照一开始我们`hexo init`生成的`.gitignore`规定的,repo里不会有`public`和`node_modules`文件夹,这些文件可以在虚拟机里生成了),然后在虚拟机里编译,生成站点以及部署到Workers.

这个过程可以让你知道,我们处理的文件中最重要的,是我们写的Markdown文章和配置文件,我们把这些东西copy似地放到虚拟机上,剩下的交给“自动化”解决

当然这只是CI的初衷之一.

[将 Hexo 部署到 Cloudflare Workers Site 上的趟坑记录](https://blog.skk.moe/post/deploy-blog-to-cf-workers-site/)

[利用 GitHub Actions 自动部署 Hexo 博客](https://sanonz.github.io/2020/deploy-a-hexo-blog-from-github-actions/)

[使用Actions+子模块+ZEIT搭建静态博客](https://blog.fun4go.top/%E9%9D%99%E6%80%81%E5%8D%9A%E5%AE%A2%E9%83%A8%E7%BD%B2%E6%96%B9%E6%A1%88.html)

[GitHub Action Docs](https://docs.github.com/cn/free-pro-team@latest/actions)

[Cloudflare Workers Docs](https://workers.cloudflare.com/docs)

[Cloudflare Wrangler Configuration](https://developers.cloudflare.com/workers/cli-wrangler/configuration)