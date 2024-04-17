---
title: Linux双系统问题杂谈（持续更新）
slug: linux-win-issue
tags: [Linux, ArchLinux, Helper]
categories: 操作系统
date: 2020-07-11T20:21:56+08:00
lastmod: 2020-07-11T20:22:28+08:00
---

![](https://cdn.jsdelivr.net/gh/hotsnow-sean/image-host/img/20200707211141.png)
玩 ArchLinux 双系统时遇到的一些小问题，记录一下

<!--more-->

## 要删除双系统时的操作（不用多余软件，方法来自网上）

1. 首先将分给 Linux 的磁盘空间在 windows 的磁盘管理中进行格式化或删除卷，如果你不用这块空间的话不删除也无伤大雅；
2. 接下来分两步，之间无顺序要求：
   1. 删除多余启动项（cmd 依次运行命令）
      - step1：管理员权限运行 `Bcdedit /enum firmware`，找到带有 Linux 字样的那一项，把它的标识符复制下来
      - step2：保存现有引导项 `Bcdedit /export savebcd`
      - step3：删除多余引导项 `Bcdedit /store savebcd /delete {2eda5080-380c-11ea-b5c6-806e6f6e6963}` 大括号里的值换成第一步记录的标识符
      - step4：导入处理好的文件 `Bcdedit /clean /import savebcd`
   2. 删除多余引导项文件（启动项被删除之后只是启动时我们看不见选项了，但是实际上启动项文件依旧在磁盘里存放着，当然也要给它干掉咯）
      - step1：打开 cmd 运行 `diskpart`
      - step2：在新窗口输入 `list disk`
      - step3：根据实际情况选择磁盘 `select disk 0`，把 0 换成启动分区所在的磁盘号
      - step4：列出所选磁盘分区 `list partition`
      - step5：选中 EFI 所在分区 `select partition 1`，把 1 换成启动分区的编号，一般是那个 100M 或 512M 大的分区
      - step5：给所选分区分配盘符 `assign letter=p`
      - step6：管理员权限打开记事本，按下打开文件选项，在此窗口中对相关文件进行删除操作
      - step7：回到 diskpart 窗口，取消盘符分配 `remove letter=p`

## Linux 和 Windows 时间不统一解决办法（二者选其一）

- case1：在 Linux 终端输入 `sudo hwclock --systohc --localtime`
- case2：在 windows 下运行 `regedit` 打开注册表，依次进入 `\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\TimeZoneInformation`，新建名为 `RealTimeIsUniversal` 的 `QWORD` 值，令其为 `1 [十六进制]`

## Linux 下键盘灯光控制

貌似只能用命令控制：

- linux 开启键盘灯光 `xset led named 'Scroll Lock'`
- linux 关闭键盘灯光 `xset -led named 'Scroll Lock'`

## linux 获取窗口类信息

忘了有什么用了，反正运行后可以看到窗口的相关信息
`xprop | awk ' /^WM_CLASS/{sub(/.* =/, "instance:"); sub(/,/, "\nclass:"); print} /^WM_NAME/{sub(/.* =/, "title:"); print}'`

## grub2 命令行启动 Linux 系统

有时候启动系统莫名奇妙卡到 grub 的命令行，这时如果 linux 系统本身完好，可以用以下命令启动：

1. `ls` 查看磁盘及分区信息，可以用来寻找 grub 所在分区
2. `set root=(hdX,X)` 选中找到的分区
3. `linux /[linux内核文件] root=/dev/sdXX` 内核文件就在所选分区内，文件名开头为 vmi，后面的 root 路径为根目录所在分区，不确定可以查看`etc/fstab`文件
4. `initrd /[init文件]` init 文件同样在所选分区，文件名开头为 init，后缀为 img
5. `boot`

_待续......_
