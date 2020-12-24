---
title: 用Github Action部署一个Worker站点
date: 2020-12-23 21:30:02
tags:
- cloudflare
- action
- hexo
categories:
- blog
---

### 缘起

苏卡卡巨佬的一篇博客

[将 Hexo 部署到 Cloudflare Workers Site 上的趟坑记录](https://blog.skk.moe/post/deploy-blog-to-cf-workers-site/)

看完这篇博客就基本上可以知道如何部署了,下面只是我的踩坑记录

### 我跳

整个部署的基本流程是这样的:先在本地完成一系列相关配置,然后配置Action完成原本在本地的部署工作

我之前写的博文很少,加之使用了一个需要配置很久的主题,从零开始配置hexo想必要比将现有的搬到云端要来得简单

新建一个目录,完成一系列初始化

    hexo init
    git init

在Github新开一个repo,然后把远程仓库设置过去

    // 这里没命令,我git只会图形界面操作和git push

对于主题,找一个简单一点的主题

如果你用的是clone后编译的主题,先clone到themes目录下

    git clone ... themes/theme-name //原来我还会git clone

这是我踩的第一个坑,注意告诉git这个主题是一个子模块,否则Action build的时候checkout部分会报错

    // Google will tell you everything,可以查询相关命令完成
    // 也可以自己新建一个`.gitmodules`文件,填入

    [submodule "hexo-theme-light"]
	path = themes/light
	url = https://github.com/tommy351/hexo-theme-light

记得配置`_config.yml`,然后`hexo g`,`hexo s`检查一下是否成功生成站点

然后是配置wrangler

当初我用这个CLI最麻烦的问题,是不知道为什么安装时下载极慢,官方支持manual安装,下载可执行文件然后扔到某个$PATH路径下

具体看skk的博客就可以了,cloudflare的命令行工具一堆emoji,总有点出戏

要注意的一点是,如果你要使用自己的域名,就得将一个二级域名交由cloudflare管理,那个zone id就是指向你的这个域名的;route也是在这种情况下才需要的参数

如果没有就是直接使用你给Workers配置的子域

配置好后,试着publish一次,通过了就可以往下走了

### 趟坑

我的这个CI部署基本上是完全模拟了本地的的情况,就是Action的虚拟机会拉取我在Github上的repo(repo里只有基本的东西,按照一开始我们`hexo init`生成的`.gitignore`规定的,不会有public和node_modules文件夹),然后在虚拟机里编译,生成站点以及部署到Worker,两个文件夹都是在那时生成的

[利用 Github Actions 自动部署 Hexo 博客](https://sanonz.github.io/2020/deploy-a-hexo-blog-from-github-actions/)

[Github Action Docs](https://docs.github.com/cn/free-pro-team@latest/actions)

[Cloudflare Workers Docs](https://workers.cloudflare.com/docs)