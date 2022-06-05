---
title: "APFS 和 TimeMachine: What I recently know"
date: 2022-06-05 23:56:00
updated: 2022-06-05 23:56:00
tags:
- macOS
categories:
- guide
desc: 最近研究了一下 macOS 的文件系统，发现 TimeMachine 的实现和它密切相关
---

之前在 Disk Utility.app 里发现 MBP 的文件系统有点复杂，于是研究一番，有了下面的笔记

<!--more-->

## APFS

### macOS 视角下的磁盘逻辑结构

首先复习一下一个硬盘的逻辑结构

```
|---------------Physical Disk---------------|
|--------------Partition Scheme-------------|
|-----Partition 0-----|-----Partition 1-----|
```

Partition Scheme 就是指 MBR 或者 GPT

而在 macOS 的视角下（或者说 macOS 的 `Disk Utility.app`/`diskutil` 的视角下）

```
|-------------------------------Physical Disk-------------------------------|
|------------------------------Partition Scheme-----------------------------|
|--------------Container 0--------------|------------Partition 1------------|
|--Volume 1--|--Volume 2--|--FreeSpace--|---Physical Volume of Partition1---|
```

当 Partition 0 被格式化为 APFS 后，它被视为一个 Container，一个 Container 下可以有多个 Volume，并且它们共享 Container 中的空闲空间，在同一个 Container 下查看 Volume 的空闲空间可以发现它们的值是一样的。

macOS 的挂载对象是 Volume 或者 Physical Volume of Partition，后者不过是对已有的硬盘逻辑结构的兼容，因为在其他类 Unix，挂载对象都是 Partition，macOS 为每个非 APFS 的 Partition 抽象出来一个 Physical Volume。将 Volume 视为挂载对象的好处显而易见，因为 Volume 的大小是动态的，如此每个 Volume 可以根据实际需要占用 Container 的空间，而非一开始就固定死了

macOS 还处理了一个问题。在原始的硬盘逻辑，硬盘(或者说包含 scheme 的硬盘)被抽象成文件，比如 disk0，partition 也被抽象成文件，比如 disk0s1。macOS 对于 Container 的抽象处理是这样的：硬盘(或者说包含 scheme 的硬盘)依旧被抽象成 disk+数字 的形式，Container 也是这么处理的，如此一来，一个 Volume 就可以被处理成 disk+数字+s+数字 的形式，符合人们对挂载对象的认识。

### 快照

下面是写下这篇文章时，我的 SSD 的逻辑结构

```
> diskutil list
/dev/disk0 (internal, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      GUID_partition_scheme                        *251.0 GB   disk0
   1:                        EFI EFI                     314.6 MB   disk0s1
   2:                 Apple_APFS Container disk1         250.7 GB   disk0s2

/dev/disk1 (synthesized):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      APFS Container Scheme -                      +250.7 GB   disk1
                                 Physical Store disk0s2
   1:                APFS Volume Macintosh HD - Data     103.1 GB   disk1s1
   2:                APFS Volume Preboot                 241.4 MB   disk1s2
   3:                APFS Volume Recovery                1.1 GB     disk1s3
   4:                APFS Volume VM                      3.2 GB     disk1s4
   5:                APFS Volume Macintosh HD            15.2 GB    disk1s5
   6:              APFS Snapshot com.apple.os.update-... 15.2 GB    disk1s5s1
```

> diskutil 命令行工具显示的信息比 Disk Utility.app 更加详细，同时也更加底层。比如使用 `diskutil info disk1s5`，它会告诉你 disk1s5 的所在的 Container 是 disk1，并且 Contianer 的物理存储为 disk0s2

这个逻辑结构除了作为上述文字的一个示例，还说明了我的 macOS 是从 APFS 快照中启动的。disk1s5s1 的类型为 APFS 快照，根据 `man diskutil` 中关于 APFS 的叙述

> However, it is also possible for `f_mntfromname` to have a 3-part form ("diskCsVsS") if you are rooted (booted) from an APFS Snapshot; in this case, its "base" Volume (e.g. "diskCsV") will not be mounted.

即有着 disk1s5s1 这种 device node，说明系统是从快照中启动的，同时快照的本体（基础 Volume）没有被挂载

关于快照，可以把它看作是某个 Volume 在某个时间点的只读备份，一个 Volume 可以没有或者拥有多个快照。快照实际上是 APFS 文件系统维护的、相对于当前 Volume 状态的额外信息，现阶段的 TimeMachine 备份就是依靠 APFS 的快照机制实现的

在 diskutil 中，使用命令 `diskutil apfs listSnapshots` 可以查看特定 Volume 的所有快照。在 macOS Monterey 中，根据[这篇教程](https://support.apple.com/guide/disk-utility/view-apfs-snapshots-dskuf82354dc/mac)，Disk Utility.app 也可以查看快照，关于里面显示的元信息，可以参照[这篇文章](https://eclecticlight.co/2021/11/13/understanding-snapshot-data-in-disk-utility/)

## TimeMachine

APFS 的快照的名字基本由两部分组成，一是创建快照的程序标识符，如

+ `com.apple.os.update`
+ `com.apple.TimeMachine`

然后是一段程序自己用于内部管理快照的信息，对于 TimeMachine，就是 日期时间+[.backup/local]，`backup` 是指创建在备份硬盘的备份快照，`local` 指创建在本地硬盘的本地快照。TimeMachine 在备份时需要先在本地硬盘创建一个快照，然后借助这个快照和备份硬盘上已有的快照比对，来往备份硬盘上写入修改信息，然后对备份磁盘进行一次快照，这样一个可供数据恢复的 TimeMachine 时间点就形成了

更具体而言，在本地创建的快照是对存储有用户数据的 Volume 进行的，往往是名为 `Macintosh HD - Data` 的 Volume。在本地形成快照后，TimeMachine 在备份硬盘的设置 TimeMachine 备份的 Volume 上寻找最近的 backup 快照，比对信息后，把新的修改往 Volume 上写，写完后，对这个 Volume 进行快照。

因此备份硬盘上的备份用 Volume 的数据就是一个精简版的 `Macintosh HD - Data`，如果不使用 Finder 而是使用其他文件管理器或者终端访问这个 Volume，只能看到最新的备份数据。Finder 在访问备份用 Volume 时做了映射（不是文件系统层面的链接），可以看到所有的备份时间点。实际上每个备份时间点都被挂载在 `/Volumes/.timemachine/[UUID]/` 下，备份用的 Volume 一旦连接到 mac 上，TimeMachine 就会为每个快照虚拟出 device node，然后挂载，可以通过 `df -h` 命令查看

关于本地硬盘的快照，TimeMachine 会自行管理，按照 [Apple 的说法](https://support.apple.com/en-us/HT204015)，在计算硬盘可用空间的时候，本地快照的大小会被计算进去，并且在硬盘需要空间时，TimeMachine 只会保留最新的本地快照

如果在 TimeMachine 中启用自动备份，则大致每隔一个小时会创建本地快照，如此，在备份硬盘连接时，可以往备份用 Volume 写入每个小时的备份数据

## Reference

[Apple Developer - About Apple File System](https://developer.apple.com/documentation/foundation/file_system/about_apple_file_system)

[View APFS snapshots in Disk Utility on Mac](https://support.apple.com/guide/disk-utility/view-apfs-snapshots-dskuf82354dc/mac)

[Understanding snapshot data in Disk Utility](https://eclecticlight.co/2021/11/13/understanding-snapshot-data-in-disk-utility/)

[About Time Machine local snapshots](https://support.apple.com/en-us/HT204015)

[Man Page for diskutil(Open in macOS)](x-man-page://diskutil)