---
title: subconverter 配置思路和代理路由软件相关介绍
date: 2022-02-19 22:25:47
updated: 2022-02-19 22:25:47
tags:
- proxy
categories:
- guide
---

配置自己部署的 subconverter 一直是件麻烦事，写这篇文章来大致记录一下思路，顺带介绍一些 macOS 上用的代理路由软件和一些零散配置

---

tindy2013/subconverter - GitHub
https://github.com/tindy2013/subconverter

---

<!--more-->

### subconverter

[Github Repo](https://github.com/tindy2013/subconverter)
[Docs_cn](https://github.com/tindy2013/subconverter/blob/master/README-cn.md)

用 C++ 写的订阅转化后端，我主要使用的功能是：

+ 在 url 中提供配置文件（称为**外部配置、config**）控制转化行为
  
  外部配置仅支持配置**规则集**和**代理组**以及**为指定的代理软件设定默认模板**

+ 模板功能

  使用了 C++ 库 INJA 来支持模板功能，从而可以在一个 tpl 文件中写出支持多个代理路由软件的框架型配置文件，如默认模板 `all_base.tpl`

+ 使用配置档案来缩短过长的 url

  使用 url parameter 来配置订阅转化会导致过长的 url，subconverter 可以使用**配置档案、profile** 来将 url parameter 写入本地文件中（截止到目前，subconverter 还不支持远程 profile），然后在 url 中使用 getprofile 来获得等价效果

将从服务提供商那里得到的订阅链接转化为自定义的代理路由软件的配置，这个过程使用 subconverter 分为三步：

1. 获得订阅链接，并 urlencode
2. 获得外部配置、config 的url，并 urlencode
3. 根据规则，拼接转化目标类型+订阅链接+外部配置+额外的配置（和 tpl 模板文件对应）

维护自己要使用的订阅转化方式，我分为两个层次

1. 维护通用的框架型配置文件 tpl，tpl 里的条件判断根据需要设置，来支持上述的第三步**额外的配置**（即模板中使用的 request 前缀，其来源为 url 中额外的 parameter）
2. 维护外部配置、config，主要维护代理组和代理规则，这个需要确保及时更新（使用 github 上的热门 ruleset repo）和自定义（使用自己 repo 里的 ruleset）

如果要使用短链接，可以使用 profile 功能，但要确保 pref.yml（subconverter 的配置文件）里 `api_mode=false` 和修改 token

### Clash
用 Go 写的路由软件
[Dev wiki](https://github.com/Dreamacro/clash/wiki)
[GitBook](https://lancellc.gitbook.io/clash/)

> 在 subconverter 中将订阅类型设定为 clash 可以生成 Clash 的配置文件，这个配置文件集路由规则和软件设定于一体

Clash 本体为命令行软件，并且提供了 RESTful API 来**简单**地控制 Clash 本身的路由行为，所以可以简单地借助 webui 来控制 Clash，详见其配置文件的 external-controller 和 external-ui 选项。同时，几乎所有 webui 只能做到简单的控制和监视

在本地设备上使用 Clash 主要是两种方式，一是配置部分软件主动访问 `127.0.0.1:[port]` 来使用代理，二是配置代理路由软件进行全局代理，但几乎所有代理路由软件的默认全局代理都有局限，尤其代理不了命令行软件，所以在 Clash Premium 中提供 TUN mode 来加强全局代理（具体的实现院里方式详见文档），但是 Clash Premium 本身并没有提供额外的开关控制 TUN，而是根据配置文件来控制

通过文件来开关 TUN 缺乏便捷性，所以各大 GUI 都提供选项开关 TUN（涉及这一点的 GUI 版本几乎都是闭源的）

#### Clash for Windows(cfw)
Clash 的跨平台 GUI 客户端（Electron 注意⚠️），在 macOS 使用 Clash 的默认配置文件夹（可以修改）作为自己的配置文件夹，注意和同类软件（包括 Clash）的冲突

支持 TUN 配置和 Mixin（为所有配置文件注入 Mixin 配置的配置块，为 cfw 自身特性）

[Github Repo](https://github.com/Fndroid/clash_for_windows_pkg)
[Docs](https://docs.cfw.lbyczf.com/)

##### Mixin 配置
尝试配置了 dns 块（后来写入了 subconverter 的模板文件里），参考了两篇博文和一份配置

1. [如何选择适合的公共 DNS？- Skk](https://blog.skk.moe/post/which-public-dns-to-use/#%E5%9B%BD%E5%A4%96%E5%B8%B8%E7%94%A8-DNS-%E6%9C%8D%E5%8A%A1)

2. [Clash 入门指南 02 DNS](https://blog.rssins.net/archives/1379)

3. [ACL4SSR](https://github.com/ACL4SSR/ACL4SSR/blob/master/Clash/GeneralClashConfig.yml)，这个配置的注释中包含对 Clash 使用 dns 配置块的行为的解释

```yaml
dns:
  enable: true
  listen: 0.0.0.0:53
  # ipv6: false # when the false, response to AAAA questions will be empty

  # These nameservers are used to resolve the DNS nameserver hostnames below.
  # Specify IP addresses only
  default-nameserver:
    - 114.114.114.114
    - 8.8.8.8
  enhanced-mode: redir-host # or fake-ip
  fake-ip-range: 198.18.0.1/16 # Fake IP addresses pool CIDR
  # use-hosts: true # lookup hosts and return IP record
  
  # Hostnames in this list will not be resolved with fake IPs
  # i.e. questions to these domain names will always be answered with their
  # real IP addresses
  # fake-ip-filter:
  #   - '*.lan'
  #   - localhost.ptlogin2.qq.com
  
  # Supports UDP, TCP, DoT, DoH. You can specify the port to connect to.
  # All DNS questions are sent directly to the nameserver, without proxies
  # involved. Clash answers the DNS question with the first result gathered.
  nameserver:
    - dhcp://en0 # dns from dhcp
    - 114.114.114.114 # default value
    - 8.8.8.8 # default value
    - https://doh.pub/dns-query
    - https://dns.alidns.com/dns-query

  # When `fallback` is present, the DNS server will send concurrent requests
  # to the servers in this section along with servers in `nameservers`.
  # The answers from fallback servers are used when the GEOIP country
  # is not `CN`.
  fallback:
    - 208.67.222.222:5353
    - https://dns.google/dns-query
    - https://1.1.1.1/dns-query

  # If IP addresses resolved with servers in `nameservers` are in the specified
  # subnets below, they are considered invalid and results from `fallback`
  # servers are used instead.
  #
  # IP address resolved with servers in `nameserver` is used when
  # `fallback-filter.geoip` is true and when GEOIP of the IP address is `CN`.
  #
  # If `fallback-filter.geoip` is false, results from `nameserver` nameservers
  # are always used if not match `fallback-filter.ipcidr`.
  #
  # This is a countermeasure against DNS pollution attacks.
  fallback-filter:
    geoip: true
    geoip-code: CN
    ipcidr:
      - 240.0.0.0/4
      - 127.0.0.1/8
      - 0.0.0.0/32
    domain:
      - '+.google.com'
      - '+.facebook.com'
      - '+.youtube.com'
  
  # Lookup domains via specific nameservers
  # nameserver-policy:
  #   'www.baidu.com': '114.114.114.114'
  #   '+.internal.crop.com': '10.0.0.1'
  ```

#### ClashX
轻量级的 macOS 客户端，支持 TUN 模式（即 Enhanced Mode，在 ClashX Premiun 版本中），但是并不是根据配置文件来设置（会被 override），而是通过 GUI 选项

[Github Repo](https://github.com/yichengchen/clashX)