---
title: grub rescue
date: 2026-01-06
summary: 在windows下搞坏grub又没有live系统，使用神秘的grub rescue启动linux
tags:
categories:
topics:
series:
---

# grub rescure

全完蛋的时候还是要live iso进去搞

## 启动

`ls` 查看分区

我用的是btrfs系统。查找 Btrfs 分区 `ls (hd0,gpt2)/`

设置prefix `set prefix=(hd0,gpt2)/@/boot/grub`  
或者，如果 @boot 是独立子卷 `set prefix=(hd0,gpt2)/@boot/grub`  
或者，如果没有子卷结构（早期安装） `set prefix=(hd0,gpt2)/boot/grub`

加载 Btrfs 模块 `insmod btrfs`

继续正常流程

```
set root=(hd0,gpt2)
insmod normal
normal
```

## 修复grub

安装

```
grub-install --target=x86_64-efi --efi-directory=/efi  --bootloader-id=ARCH
```

配置

```
grub-mkconfig -o /boot/grub/grub.cfg
```

## 其他

### **核心命令**

```bash
# 1. ls - 列出设备和分区
ls                    # 列出所有设备
ls (hd0)             # 查看第一块磁盘
ls (hd0,1)           # 查看第一块磁盘的第一个分区
ls (hd0,gpt1)/       # 查看分区内容

# 2. set - 查看/设置环境变量
set                   # 查看所有变量
set root=(hd0,1)     # 设置根设备
set prefix=(hd0,1)/boot/grub
```

### **模块相关命令**

```bash
# 3. insmod - 加载模块
insmod normal        # 加载 normal 模块
insmod linux         # 加载 Linux 引导模块
insmod chain         # 加载链式引导模块
insmod part_msdos    # MBR 分区支持
insmod part_gpt      # GPT 分区支持
insmod ext2          # ext2/3/4 支持
insmod btrfs         # Btrfs 支持
insmod ntfs          # NTFS 支持
insmod fat           # FAT/FAT32 支持
insmod xfs           # XFS 支持
insmod configfile    # 配置文件支持

# 4. lsmod - 列出已加载模块
lsmod
```

### **引导相关命令**

```bash
# 5. normal - 进入正常模式
normal              # 如果加载了 normal 模块

# 6. boot - 启动系统
boot               # 启动已加载的系统

# 7. linux/initrd - 手动引导 Linux
linux (hd0,2)/vmlinuz-linux root=/dev/sda3
initrd (hd0,2)/initramfs-linux.img
boot
```

### **磁盘和分区侦察**

```bash
# 查看所有存储设备
ls

# 典型输出示例：
# (hd0) (hd0,msdos1) (hd0,msdos2) (hd0,msdos3)
# 或者 UEFI/GPT：
# (hd0) (hd0,gpt1) (hd0,gpt2) (hd0,gpt3)

# 探查每个分区的内容
ls (hd0,1)/
ls (hd0,2)/
ls (hd0,3)/
```

### **识别关键分区特征**

```bash
# 查找 Linux 分区特征
ls (hdX,gptY)/boot/          # 查找 /boot 目录
ls (hdX,gptY)/etc/           # 查找 /etc 目录
ls (hdX,gptY)/usr/           # 查找 /usr 目录

# 查找 GRUB 文件
ls (hdX,gptY)/boot/grub/     # GRUB 目录
ls (hdX,gptY)/grub/          # 独立 /boot 分区

# 查找 Windows 分区特征
ls (hdX,msdos1)/Windows/     # Windows 系统目录
ls (hdX,msdos1)/Users/       # Windows 用户目录
```

### **文件系统特定探查**

```bash
# 先加载文件系统模块再查看
insmod ext2
ls (hd0,1)/

insmod btrfs
ls (hd0,2)/           # 可能看到 @, @home 等子卷

insmod ntfs
ls (hd0,3)/           # Windows 分区

insmod fat
ls (hd0,gpt1)/        # EFI 系统分区
```

### **简单的 GRUB 文件丢失**

```bash
set prefix=(hd0,gpt2)/boot/grub
set root=(hd0,gpt2)
insmod normal
normal
```

进入系统后运行：grub-install /dev/sda

### **分区编号改变**

原来的 (hd0,2) 现在变成了 (hd0,3)

```bash
set prefix=(hd0,gpt3)/boot/grub
set root=(hd0,gpt3)
insmod normal
normal
```

进入系统后更新 /etc/fstab 和 GRUB

### **手动引导进入系统**

```bash
# 1. 找到内核和 initramfs
ls (hd0,gpt2)/boot/
# 应该看到 vmlinuz-linux 和 initramfs-linux.img

# 2. 加载必要模块
insmod part_gpt
insmod ext2
insmod gzio
insmod xzio
insmod lzopio

# 3. 设置根设备
set root=(hd0,gpt2)

# 4. 加载内核和 initramfs
linux /boot/vmlinuz-linux root=/dev/sda2 rw
# 或者使用 UUID
linux /boot/vmlinuz-linux root=UUID=xxxx-xxxx rw

initrd /boot/initramfs-linux.img

# 5. 启动
boot
```

### **引导 Windows 系统**

找到 Windows 分区
对于 UEFI

```bash
insmod part_gpt
insmod fat
ls (hd0,gpt1)/EFI/Microsoft/Boot/bootmgfw.efi
chainloader /EFI/Microsoft/Boot/bootmgfw.efi
boot
```

### **调试模式**

```bash
# 进入 GRUB 时按 ESC 或 C
# 或者从 rescue 模式
set debug=all
# 查看详细的调试信息
```

### **查看环境变量**

```bash
set
# 重要变量：
# prefix - GRUB 文件路径
# root - 根设备
# cmdpath - GRUB 镜像路径
# debug - 调试级别
```

### **修复常见错误**

#### **unknown filesystem**

```bash
# 加载对应的文件系统模块
insmod ext2    # 对于 ext2/3/4
insmod btrfs   # 对于 Btrfs
insmod xfs     # 对于 XFS
insmod fat     # 对于 FAT/EFI 分区
insmod ntfs    # 对于 NTFS
```

#### **out of memory**

```bash
# 清除已加载的模块
rmmod 模块名

# 按需加载最小模块集
insmod part_gpt
insmod ext2
insmod linux
```

### **创建 GRUB 修复盘**

```bash
# 创建一个 GRUB 救援 USB
dd if=/usr/lib/grub/i386-pc/boot.img of=/dev/sdX bs=446 count=1

# 或者使用更完整的方法
grub-install --target=i386-pc --boot-directory=/mnt/usb/boot /dev/sdX
```

### **配置 GRUB 更健壮**

编辑 `/etc/default/grub`：

```bash
# 使用 UUID 而不是设备名（更稳定）
GRUB_DISABLE_LINUX_UUID=false

# 启用 OS 探测
GRUB_DISABLE_OS_PROBER=false

# 设置备用引导路径
GRUB_BACKUP_PATHS="/boot/grub /grub"

# 预加载必要模块
GRUB_PRELOAD_MODULES="part_gpt part_msdos ext2 btrfs xfs fat ntfs"
```

### **八、特殊文件系统注意事项**

#### **ZFS**

```bash
# 需要额外模块
insmod zfs
set root=ZFS=poolname/dataset
```

#### **LVM**

```bash
insmod lvm
ls (lvm/volume_group-logical_volume)/
```

#### **加密分区（LUKS）**

```bash
# 需要先解密
cryptomount (hd0,gpt2)
set root=(crypto0)
```

## **常用命令速查表**

| 命令          | 用途          | 示例                   |
| ------------- | ------------- | ---------------------- |
| `ls`          | 列出设备/分区 | `ls (hd0,1)/`          |
| `set`         | 设置变量      | `set root=(hd0,1)`     |
| `insmod`      | 加载模块      | `insmod ext2`          |
| `normal`      | 进入正常模式  | `normal`               |
| `linux`       | 加载内核      | `linux /vmlinuz`       |
| `initrd`      | 加载initrd    | `initrd /initramfs`    |
| `boot`        | 启动系统      | `boot`                 |
| `configfile`  | 加载配置      | `configfile /grub.cfg` |
| `chainloader` | 链式引导      | `chainloader +1`       |
