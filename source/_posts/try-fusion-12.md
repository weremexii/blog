---
title: 获取 Fusion 12 序列号
date: 2020-12-23 20:59:39
updated: 2020-12-23 20:59:39
tags: 
- emulator
- macOS
categories: 
- guide
desc: 获取 Fusion 12 序列号
---

可以白嫖的 macOS 虚拟机软件

<!--more-->

### 缘起

在 macOS 上使用虚拟机软件，不外乎是 Parallels，Fusion 或者 VisualBox。Paralles 以流畅到极致的 Windows 体验著称，但虚拟的其他系统体验却相形见绌。VisualBox 则被称为虚拟拖拉机，体验可见一斑（然而博主却在类似 MuMu 模拟器之类的 Android 虚拟机软件中看到 VisualBox）。虚拟 Linux 系系统就选 Fusion了。

碰巧的是，VMware 在 8 月[发布 Fusion 12](https://blogs.vmware.com/teamfusion/2020/08/announcing-fusion-12-and-workstation-16.html)，并更新它的授权体系，允许免费的个人使用。这意味这博主不再需要 cracked version 啦，可以名正言顺地白嫖啦。

PS:发布博文后，过了大概半个月才给出[下载地址](https://my.vmware.com/web/vmware/downloads/info/slug/desktop_end_user_computing/vmware_fusion/12_0)。

正文开始

### 获取序列号

即便 VMware 放出个人版免费使用，也逃不了注册账户才能获得软件使用的序列号（安装的 fusion 12 有试用期，必须输入序列号才能解除试用）。值得注意的是，VMware的产品授权有着类似于 AppStore 的地区区别（国区美区这样的）。仔细来讲，一个账户可以拥有同时不同地区同一个软件的授权，地区看的是你登录 VMware 时的 IP 地址。

博主在这里被坑了一波，我默认的代理节点为香港，通过代理登录 VMware 账户，点击`上述`下载页面的 `Get Your License Key` 再点击 `My VMware License Key`，会跳转到 vSphere Hypervisor 的授权界面。几番尝试，博主关掉代理重新登录（建议退出登录后清除 cookie 再登录）,重复上述操作，跳转到了 404。看来不同地区优惠不一样啊，抱着这样的想法，博主挂上了美国的代理。

当然，在美国代理下上上述操作也不一定能拿到序列号，毕竟你并没有拿过序列号，My License Key 中不一定会有。所幸博主在一个[新闻页面](https://www.macrumors.com/2020/09/15/vmware-fusion-12-available/)找到了 Fusion 12 的[授权地址](https://my.vmware.com/web/vmware/evalcenter?p=fusion-player-personal)（类似于上述的那个 vSphere Hypervisor），点击 Registrate，获得序列号（请全程美国代理）