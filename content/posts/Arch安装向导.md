---
title: Arch安装向导
slug: arch-install-guide
tags: [Linux, ArchLinux, Helper]
categories: 操作系统
date: 2020-07-06T07:32:52+08:00
lastmod: 2020-08-19T07:32:45+08:00
---

![](https://cdn.jsdelivr.net/gh/hotsnow-sean/image-host/img/20200707211141.png)
2020-08-19 更新添加问题解决章节  
2020-07-30 更新最新联网方式以及修正部分内容

---

之前玩 Linux 时记录的文档，托管到博客上做记录  
基本根据[ArchLinux 官方向导 <i class="fa fa-external-link-alt"></i>](https://wiki.archlinux.org/index.php/Installation_guide "官方安装向导")而来，综合了一些会遇到的问题

<!--more-->

{{< note type="danger" >}}
注意本文更新时间，部分内容可能已经不适用
{{< /note >}}

## 一、 准备事项

### ~~1、连接网络（根据情况选择以下三种方法之一）~~

- 准备工作：使用 `ip link` 命令查看网卡状态，若网卡未开启，使用命令 `ip link set up `_`interface`_ 开启

#### 1.1 使用 `wifi-menu` 连接无线

- 这种方法看不到隐藏网络

#### 1.2 连接有线网络

- 有线网络只需准备好后，只需要分配 IP，使用命令 `dhcpcd &`

#### 1.3 使用 `wpa_supplicant` 连接无线

- a. 生成网络配置文件：`wpa_passphrase `_`MYSSID`_` `_`PASSWORD`_` > internet.conf`
- b. 若为隐藏网络，编辑生成的配置文件，加入一行 `scan_ssid=1`
- c. 调用配置文件连接网络：`wpa_supplicant -B -i `_`interface`_` -c internet.conf &`
- d. 分配 IP：`dhcpcd &`

### 1、连接网络（2020 年 7 月份之后的安装镜像采用的新方法）

- 查看网卡信息等依旧可以采用上边提到的 `ip link` 等方法。
- 具体网络连接只需要采用一个工具：[`iwctl` <i class="fa fa-external-link-alt"></i>](https://wiki.archlinux.org/index.php/Iwctl)，简单易用，轻松应对有线无线隐藏等各种情况。

### 2、同步时间

- `timedatectl set-ntp true`

## 二、 分区并挂载

- 使用 `fdisk -l` 查看目前分区情况，并记下需要分区的磁盘号
- 命令 `cfdisk /dev/sda` 其中/dev/sda 为需要分区的磁盘
- 分区：按照喜好大小分配即可，此次分区为 根目录 48G，家目录 68G，交换分区 4G
- 格式化
  - `mkfs.fat -F32 /dev/sdaX` 已经有 EFI 分区则不要这条命令
  - `mkfs.ext4 /dev/sda1` 建立根目录及家目录格式
  - `mkswap /dev/sda2` 建立交换分区及开启交换分区
  - `swapon /dev/sda2`
- 挂载
  - `mount /dev/sda1 /mnt`
  - `mount /dev/sda3 /mnt/home`
  - `mount /dev/sdaX /mnt/boot`

## 三、 修改软件源

- `vim /etc/pacman.d/mirrolist` 将其中想要使用的源提到最前面，这步在第四步之前均可

## 四、 安装基础包

- `pacstrap /mnt base linux linux-firmware`（最好查看官方 wiki，防止更新变化）
- `pacstrap /mnt base-devel` 可选
- 配置 fstab 文件 `genfstab -U /mnt >> /mnt/etc/fstab`

## 五、 进入/mnt 并配置

- `arch-chroot /mnt`
- 更改时区
  - `ln -sf /usr/share/zoneinfo/Region/City /etc/localtime`
  - `hwclock --systohc`
- 语言配置编辑
  - `vim /etc/locale.gen` 去掉要使用语言前的注释
  - `vim /etc/locale.conf` 输入 `LANG=en_US.UTF-8`
  - 运行 `locale-gen`
- 安装网络包

  - ~~`pacman -S wpa_supplicant dhcpcd `~~
  - 【修正】只需安装 `networkmanager` ，然后运行 `systemctl enable NetworkManager.service` 让其开机自启即可。无需安装之前的两个包，防止冲突

- 网络配置
  - `在 /etc/hostname 文件中输入 myhostname`
  - `vim /etc/hosts` 输入以下内容
    ```
    127.0.0.1	localhost
    ::1		    localhost
    127.0.1.1	myhostname.localdomain	myhostname
    ```
- `passwd` 设置 root 用户密码
- 安装引导程序
  - `pacman -S grub efibootmgr os-prober intel-ucode`
  - `grub-install --target=x86_64-efi --efi-directory=esp --bootloader-id=GRUB`
  - `grub-mkconfig -o /boot/grub/grub.cfg`

## 六、 退出并重启

- `exit` 退出 `/mnt`
- `umount -R /mnt`

## 七、 桌面环境安装

### 1、创建普通用户

- 创建并添加用户到组：`useradd -m -G wheel -s login_shell `_`username`_
- 设置密码：`passwd username`
- 开启 sudo 命令权限：`visudo` 取消 wheel 行注释

### 2、开启 multilib 软件库

- `vim /etc/pacman.conf` 取消对应注释

### 3、安装图形服务

- `pacman -S xorg`

### 4、安装显卡驱动（根据硬件查找官网）

- intel：`xf86-video-intel` ~~`lib32-virtualgl`~~
- OpenGL：`mesa` ~~`nvidia-390xx-utils`~~
- nvidia：`nvidia-390xx` ~~`lib32-nvidia-390xx-utils`~~ 新显卡直接使用 `nvidia` 即可
- ~~`bumblebee`~~ 存在性能问题，弃用
- ~~添加用户到组：`gpasswd -a user bumblebee`~~
- ~~开启服务：`bumblebeed.service`~~
- 此时双显卡选择采用 [官方提到的其它解决方案 <i class="fa fa-external-link-alt"></i>](https://wiki.archlinux.org/index.php/NVIDIA_Optimus) 中的一个合适的即可

### 5、安装 KDE

- `pacman -S plasma kde-applications`
- 检查 `sddm`, `sddm-kcm`
- 开启服务 `systemctl enable sddm`

### 6、杂项安装

- 中文字体：`pacman -S ttf-dejavu wqy-microhei`
- 网络
  - `pacman -S networkmanager plasma-nm`
  - ~~`systemctl enable NetworkManager`~~

## 八、使用中可能的问题

### 1、ArchlinuxCN 仓库使用问题

在使用 ArchlinuxCN 中文社区仓库时，除了添加相应的服务器地址，还需要安装 `archlinuxcn-keyring` 包来导入 `GPG key`，安装时，可能会遇到很多 ERROR 报错。这时可以参考 [密钥环这篇文](https://www.archlinuxcn.org/gnupg-2-1-and-the-pacman-keyring/)， 以 root 权限用以下命令解决：

```
pacman -Syu haveged
systemctl start haveged
systemctl enable haveged

rm -fr /etc/pacman.d/gnupg
pacman-key --init
pacman-key --populate archlinux
pacman-key --populate archlinuxcn
```

运行完这些命令后再去尝试安装 `archlinuxcn-keyring` 包，应该就不会报错了。
