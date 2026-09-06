---
title: Linux 开机及引导过程
date: 2026-09-05
summary: uefi, grub 等
tags:
categories:
topics:
series:
genres:
status:
ai: true
---

# Linux 开机及引导过程

## 启动流程与核心概念

### UEFI启动全流程（四阶段）

1. **硬件初始化（UEFI固件）**：读取主板BIOS设置，扫描硬盘。
2. **UEFI Boot Manager**：读取NVRAM中的`BootOrder`，在ESP分区找到并加载`.efi`引导文件（如`grubx64.efi`或`BOOTX64.EFI`）。
3. **引导管理器（GRUB/Limine）**：加载文件系统驱动，读取Btrfs分区中的`vmlinuz-linux`（内核）和`initramfs-linux.img`到内存。
4. **Linux内核启动**：内核解压并运行`initramfs`中的`/init`脚本，加载驱动，挂载Btrfs根子卷`@`，最终移交控制权给`systemd`。

### `efibootmgr`的作用

- 是**修改主板NVRAM启动项**的管理工具，用于注册、删除或调整UEFI启动顺序。
- GRUB在安装时会自动调用它
- 它是控制“电脑默认启动谁”的核心工具

### `vmlinuz-linux` 与 `initramfs-linux.img`

- **`vmlinuz-linux`**：即Linus Torvalds维护的Linux内核本体，负责进程调度、内存管理、系统调用等核心功能。
- **`initramfs-linux.img`**：一个临时根文件系统，包含内核模块（如Btrfs、NVMe驱动）和启动脚本，用于在挂载真实根分区前加载必要驱动。
- **两者关系**：内核启动时先运行`initramfs`，由其加载驱动并挂载Btrfs分区，最后`switch_root`到真实根目录。

### `mkinitcpio` 与 `dracut`

- **`mkinitcpio`**：Arch Linux默认的`initramfs`生成工具，基于钩子（Hook）机制，配置简单，生成镜像体积小。
- **`dracut`**：Red Hat生态的标准工具，模块化、自动化程度高，对复杂场景（如LUKS+TPM）支持更好。
- **选择建议**：Arch用户默认使用`mkinitcpio`即可，无需切换到`dracut`。
- **内核更新时的行为**：`pacman -Syu`更新内核时，会自动触发`mkinitcpio`为每个内核（如`linux`和`linux-zen`）重新生成对应的`initramfs`，确保内核与模块匹配。

### UKI（统一内核镜像）

- **定义**：将内核、`initramfs`、内核命令行签名打包成一个独立的`.efi`文件。
- **优势**：启动更快、安全性更高（可整体签名），可减少对GRUB等引导器的依赖。
- **劣势**：灵活性降低（内核参数固化），更新需要重新构建，ESP分区占用空间更大。
- **与Btrfs快照的兼容性**：UKI本身**不直接支持快照**，需配合原子升级（每个快照生成独立UKI）或内核参数法（同一UKI通过`rootflags=subvol=`启动不同快照）等策略使用。
- **建议**：对于当前需求，UKI会增加管理复杂度，**暂不建议采用**。


## 引导器选择：GRUB vs Limine

### GRUB与Limine的核心区别

- **GRUB**：功能全面，支持丰富的文件系统和救援模式，配置由`grub-mkconfig`自动生成，对新手友好。
- **Limine**：设计精简、启动速度快，配置基于手写的`limine.conf`，更简洁，但不具备GRUB的救援模式。

### Limine的配置文件（`limine.conf`）示例

- **Arch Linux启动项（Btrfs子卷）**：
  ```
  :Arch Linux
      protocol: linux
      kernel_path: boot():/vmlinuz-linux
      module_path: boot():/initramfs-linux.img
      cmdline: root=UUID=你的BTRFS分区UUID ro rootflags=subvol=@
  ```
- **Windows 11启动项（链式加载）**：
  ```
  :Windows 11
      protocol: efi_chainload
      image_path: boot():/EFI/Microsoft/Boot/bootmgfw.efi
  ```

### Limine的配置文件查找顺序

- `BOOTX64.EFI`会按以下顺序扫描`limine.conf`：
  1. 与`BOOTX64.EFI`同目录（如`/efi/arch-limine/limine.conf`）
  2. `/EFI/BOOT/limine.conf`
  3. `/boot/limine.conf`
- **注意**：若在高优先级路径存在旧配置文件，会导致修改`/boot/limine.conf`无效。

### Limine相关工具

- **`limine-install`**：由官方`limine`包提供，用于部署引导代码。
- **`limine-mkinitcpio-hook`**：**非必须**。它用于自动更新带版本号的UKI或处理Secure Boot。对于固定路径（`/boot/vmlinuz-linux`）的标准用法，该钩子是多余的，甚至可能在回滚时造成混乱。
- **`efibootmgr`**：**强烈建议安装**。Limine在UEFI下默认不注册NVRAM启动项，需手动使用`efibootmgr`将Limine设为第一启动项，否则电脑可能直接进Windows。
