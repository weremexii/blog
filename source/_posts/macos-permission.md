---
title: 记macOS下Homebrew的一个权限报错问题
date: 2021-01-12 16:45:23
tags:
- macOS
- dscl
- Homebrew
categories:
- guide
desc: Homebrew安装nodejs报错/usr/local/share/systemtap/tapset is not writable
---

(´；ω；｀)

<!--more-->

### 缘起

在收拾mac上的环境配置信息的时候,我发现之前我用安装包安装了`nodejs v12`,而现在运行code-server的nodejs版本为`v15`

把一堆东西删掉之后,重新用`Homebrew`安装nodejs,报错如下

```bash
Error: Could not symlink share/systemtap/tapset/node.stp
/usr/local/share/systemtap/tapset is not writable.
```

### 解决

首先是一些基本信息

```bash
$ ls -al /usr/local/share/systemtap/
drwxr-xr-x   3 root  wheel  102 Oct 23 18:06 .
drwxrwxr-x  15 root  wheel  510 Oct 24 20:24 ..
drwxr-xr-x   2 root  wheel   68 Oct 24 20:17 tapset
```

`tapset`目录的所有者是`root`,所有组是`wheel`,原始读写权限是`755`

网上提供了很多种解决办法,毕竟这只是一个权限问题.不过为了把对系统的影响降到最小,我选择把自己加入wheel用户组并修改组的读写权限后执行安装,然后再恢复回来,而不是直接修改目录的所有者

1. 将自己(所用的用户)写入wheel组

    ```bash
    $ sudo dscl . -append /Group/wheel GroupMembership [username]
    ```

    和其他Unix-like系统不同,macOS的用户(组)管理使用`dscl`命令

    username可以使用`whoami`命令查询

2. 利用root权限(sudo执行命令)修改所有组的读写权限

    ```bash
    $ sudo chmod -R 775 /usr/local/share/systemtap/
    ```

3. 执行`brew link node`
4. 将自己从所有组删除

    ```bash
    $ sudo dscl . -delete /Group/wheel GroupMembership [username]
    ```
5. 恢复文件夹的原始读写权限

    ```bash
    $ sudo chmod -R 755 /usr/local/share/systemtap/
    ```

把这些执行完问题就解决了,不过我后来才发现,其实我(`[username]`)本来就在`wheel`用户组里,上面的步骤中只要执行`2,3,5`步即可

### Reference

[Trouble install node.js with homebrew - Stack Overflow](https://stackoverflow.com/questions/31374143/trouble-install-node-js-with-homebrew)

[mac下通过dscl命令对用户/用户组进行增删改查操作](https://segmentfault.com/a/1190000012973654)
