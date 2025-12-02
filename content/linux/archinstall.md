---
title: 双系统安装archlinux
date: 2023-10-09
summary: 双系统安装archlinux，btrfs分区、grub启动项
tags:
  - archlinux
  - geek
categories: handbook
---

# Basic installation of arch linux

> [!important] 有能力者建议看[官方安装文档](https://wiki.archlinux.org/title/Installation_guide)

双系统安装archlinux，使用btrfs文件系统。重装系统的参考。

> [!tip] windows部分
> 由于windows安装和更新可能会改动efi分区，建议先安装windows，然后再安装archlinux。  
> 如果需要干净的windows，可以参考[这里](/dev/rewindows)。

## 启动盘

下载archlinux的iso文件。找一个U盘，用 `ventoy`安装，然后把iso放进去。

可以先在win下用 <kbd>win</kbd>+<kbd>x</kbd>、<kbd>k</kbd> 打开磁盘管理，然后手动分区，用win的GUI代替之后的命令行分区。可以使用`diskgenuis`软件做更多操作。

插上U盘，重起电脑，在开机时按 <kbd>F11</kbd> <kbd>F12</kbd> <kbd>F2</kbd> 之类能进入选择启动位置的界面，选择从U盘启动。之后应该会启动U盘里的`ventoy`，选择arch的iso。这样就进入`arch live` 的系统，这个系统是用来安装arch的。

## 处理磁盘

> [!tip]
> 此时进入的是`arch live`环境，其实这个时候已经可以拔掉U盘了。在这个环境中做的修改不会保存。

接下来要预处理实际电脑上的磁盘，也就是系统安装位置。

### network and time

首先是把`arch live`系统联网，设置时间，配置软件源。使用 `iwctl` 连接网络。

```sh
iwctl # 进入iwctl
[iwd] help # 查看使用方法
[iwd] station wlan0 connect **SSID** # example
[iwd] exit # 退出iwctl
```

同步时间

```sh
timedatectl set-ntp true
timedatectl status
```

在 `/etc/pacman.d/mirrorlist`中添加pacman的源，推荐清华源。否则可能下载速度慢。

```text
Server = https://mirrors.tuna.tsinghua.edu.cn/archlinux/$repo/os/$arch
```

### 分区

> [!warning]
> 这一步是操作实际电脑的磁盘，会删除数据。请小心操作。
> 将预留空间格式化为btrfs文件系统，并分区。

首先查看分区：

```sh
fdisk -l
```

看清楚各个分区，以下以`nvme0n1p5`为例作为linux分区。

```sh
mkfs.btrfs  -L btrfs-arch /dev/nvme0n1p5 # -L means label, can be omitted
```

先挂载这个分区，根据需要创建其子卷。

```sh
mount -o /dev/nvme0n1p3 /mnt # 这条命令将分区挂载到btrfs的顶级子卷，即 / 子卷， id=5
# 通常这些子卷
btrfs subvolume create /mnt/@   # 创建顶级子卷下的 @ 子卷，将作为新linux的 / 目录
btrfs subvolume create /mnt/@home # 创建顶级子卷下的 /@home 子卷，将作为新linux的 用户家目录，即 /home
# 以下可选
btrfs subvolume create /mnt/@log
btrfs subvolume create /mnt/@cache
btrfs subvolume create /mnt/@opt
btrfs subvolume create /mnt/@tmp
btrfs subvolume create /mnt/@root
```

检查一下，应该有以下输出：

```sh
btrfs subvolume list /mnt
ID 256 gen 405 parent 5 top level 5 path @
ID 257 gen 409 parent 5 top level 5 path @home
```

> [!note]
> ID 是子卷的 id，gen 是子卷的代数，parent 是父卷的 id，top level 是顶级子卷的 id，path 是子卷的路径  
> 应该还有其他子卷的输出，这里没有列出。

接下来先取消挂载顶级子卷，再挂载子卷到对应的目录：

```sh
umount /mnt

mount -o noatime,nodiratime,compress=zstd,subvol=@ /dev/nvme0n1p5 /mnt #挂载 @ 子卷到新系统的 / 目录
mkdir -p /mnt/{boot/efi,home,var/{log,cache}} #创建新系统的 /boot/efi、/home、/var/log、/var/cache
mount -o noatime,nodiratime,compress=zstd,subvol=@home -m /dev/nvme0n1p5 /mnt/home # -m 创建挂载的文件夹
# 如果创建了对应子卷，那么
mount -o noatime,nodiratime,compress=zstd,subvol=@log -m /dev/nvme0n1p5 /mnt/var/log
mount -o noatime,nodiratime,compress=zstd,subvol=@cache -m /dev/nvme0n1p5 /mnt/var/cache
mount -o noatime,nodiratime,compress=zstd,subvol=@tmp -m /dev/nvme0n1p5 /mnt/tmp
mount -o noatime,nodiratime,compress=zstd,subvol=@opt -m /dev/nvme0n1p5 /mnt/opt
mount -o noatime,nodiratime,compress=zstd,subvol=@root -m /dev/nvme0n1p5 /mnt/root
```

查看挂载情况

```sh
findmnt -nt btrfs
```

挂载efi分区。因为单硬盘安装双系统，所以使用windows系统原有的efi分区。

```sh
mount /dev/nvme0n1p1 /mnt/boot/efi
```

> [!tip]
> win初始的efi分区大小可能不够 `/boot` ，所以可以只挂载 `/boot/efi`。也可以创建`/efi`  
> 在win下可以用 `diskgenuis` 查看efi分区。

> [!note] update 2025-07-27
> ESP分区的挂载，现在推荐直接挂载到`/efi`而不是`/boot/efi`
>
> ```sh
> mount /dev/nvme0n1p1 /mnt/efi
> ```
>
> 除此之外，`btrfs`格式的`/boot`会导致一些奇怪的报错，比如稀疏文件的问题等，以及`grub`无法设置为从上次启动的选项启动。所以也可以单开一个分区来挂载`/boot`.
> 参见[这篇博客](https://silvertuanzi.github.io/2025/02/11/arch-cinnamon/)

```shell
mkdir -p /mnt/efi
mount /dev/nvme0n1p1 /mnt/boot/efi
```

可以把这些子卷关闭写时复制

```
chattr +C /mnt/tmp
chattr +C /mnt/var/cache
chattr +C /mnt/var/log
```

## 安装系统

这一步是实际安装系统到电脑上。
install and genfstab 安装系统并生成分区表

安装系统到新分区，这里使用 `pacstrap` 命令，安装一些必要组件和后续使用的基础软件，包括linux内核、linux固件、btrfs-progs管理btrfs文件系统、dhcpcd、iwd、networkmanager管理网络、neovim编辑器、git、sudo、grub、os-prober (查找硬盘上其他系统，在grub界面启动windows系统)、efibootmgr

```sh
pacstrap -K -M /mnt base base-devel linux linux-headers btrfs-progs iwd neovim git
sudo grub os-prober efibootmgr amd-ucode dhcpcd
```

> [!note]
> 根据cpu安装 `amd-ucode` 或 `intel-ucode`

接下来生成分区表，记录这个磁盘分区是如何使用的。

```
genfstab -U /mnt >> /mnt/etc/fstab
```

## 配置grub启动

配置grub，使得电脑启动时能找到这个系统。这一步需要进入新安装的系统中：

```
arch-chroot /mnt
```

这个命令之后就进入了刚刚安装的archlinux系统，之后的命令都是在这个系统中执行的。

> [!note]
> 对于双系统安装，要让grub发现其他系统。在`/etc/default/grub`中取消注释
>
> ```
> # GRUB_DISABLE_OS_PROBER=false
> ```
>
> 对于btrfs，似乎需要一些操作才能使`os-prober`起效。  
> 在`/etc/mkinitcpio.conf` 添加 btrfs 到 MODULES=(...)行 找到 HOOKS=(...)行，更换fsck为btrfs 最终你看到的/etc/mkinitcpio.conf文件格式为
>
> ```text
> ...
> MODULES=(btrfs)
> HOOKS=(base udev autodetect modconf block filesystems keyboard btrfs)
> ...
> ```
>
> 然后重新生成 initramfs
>
> ```sh
> mkinitcpio -P
> ```

安装启动器`grub`到efi分区，并生成grub文件：

```sh
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=arch --recheck
```

```
grub-mkconfig -o /boot/grub/grub.cfg
```

> [!tip]
> 可以先对grub做一些配置，也可以后续执行。  
> 去掉 quiet 参数，调整 loglevel 值为 5 ，加入 nowatchdog 参数
>
> ```sh
> sed -i "s|GRUB_CMDLINE_LINUX_DEFAULT.*|GRUB_CMDLINE_LINUX_DEFAULT=\"loglevel=5 nowatchdog\"|" /etc/default/grub
> ```

理论上现在可以重启电脑，选择从硬盘启动，就可以看到grub界面，选择archlinux启动。但最好先做一些基本配置。

## 基本配置

首先设置root密码

```
passwd root
```

设置密码是静默输入，不会显示密码，输入两次即可。

配置locale、hostname、网络等

```sh
ln -sf /usr/share/zoneinfo/Region/City /etc/localtime #替换Region/City为你所在区域，例如Asia/Shanghai
hwclock --systohc # 同步硬件时钟
timedatectl set-ntp true
timedatectl set-local-rtc true #同步双系统时间
echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen
locale-gen
echo "LANG=en_US.UTF-8" > /etc/locale.conf #或者编辑文件，取消注释
systemctl enable NetworkManager.service # 启动网络服务
hostnamectl set-hostname <yourhostname> # 设置主机名
```

时间格式也可以设置

```sh
echo "LC_TIME=C.UTF-8" >> /etc/locale.conf # >> 表示追加。
```

hosts:

```
# /etc/hosts
127.0.0.1   localhost
::1         localhost
127.0.1.1   <yourhostname>.localdomain    <yourhostname>
```

## quit and reboot

退出新系统，取消挂载，重启

```
exit #退出chroot
umount /mnt/boot/efi
umount /mnt/home
umount /mnt
reboot
```

之后有一些[常见的配置](/linux/archpostinstall)。
