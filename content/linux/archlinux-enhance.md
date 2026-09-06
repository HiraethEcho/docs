---
title: Archlinux 性能优化
date: 2026-08-26
summary: 和 deepseek 对话，查 archwiki，以及实践后的记录
tags:
  - archlinux
  - geek
categories: handbook
series: archlinux
status: wip
ai: true
---

# Archlinux 性能优化

在优化我的 archlinux 时和 deepseek 的聊天记录汇总

## Arch Linux Btrfs 性能优化及 Swap/Zram 配置总结

### 挂载选项优化建议

- **核心调整**：去除冗余的 `nodiratime`（因 `noatime` 已隐含），保留 `noatime`；将 `compress=zstd:3` 降为 `zstd:1` 或 `zstd:2` 以平衡 CPU 开销，对日志、缓存等频繁写入目录可设为 `compress=no`；考虑对 Docker、Libvirt 等目录使用 `chattr +C` 禁用写时复制（CoW）以提升随机写入性能；可选择性添加 `autodefrag` 减少小文件碎片。
- **挂载选项说明**：`discard=async`（异步 TRIM）、`space_cache=v2`（改进空闲空间管理）均推荐保留；`ssd` 选项可保留或删除（Btrfs 会自动检测 SSD）。
- **生效方式**：修改 `/etc/fstab` 后执行 `sudo mount -o remount /` 或 `sudo mount -a` 使更改生效。

### Btrfs Swapfile 的创建与挂载

- **创建独立子卷**：用户选择创建顶层 `@swap` 子卷并挂载到 `/swap`，而非在 `@` 内部创建 `/var/swap`。操作步骤：挂载顶层卷（ID 5）创建 `@swap`，并在 `/etc/fstab` 中添加挂载项 `UUID=... /swap btrfs ... subvol=/@swap 0 0`。
- **生成 swapfile**：使用新版 `btrfs-progs` 提供的 `btrfs filesystem mkswapfile --size 4g --uuid clear /swap/swapfile` 命令，该命令自动禁用 CoW 和校验和，比手动 `fallocate`+`mkswap` 更安全。
- **开机自动激活**：仍需在 `/etc/fstab` 中添加一行 `/swap/swapfile none swap defaults 0 0`（或 `sw,pri=10`），使系统启动时自动执行 `swapon`。

### Zram 与 Swapfile 协同配置

- **Zram 简介**：在内存中创建压缩块设备用作 Swap，速度极快，但消耗 CPU，不支持休眠。
- **配置要点**：使用 `zram-generator` 时在配置中指定 `swap-priority=100`；在 fstab 中为 swapfile 添加 `pri=10`；同时必须禁用 Zswap（内核参数 `zswap.enabled=0`），避免冲突。
- **休眠兼容**：系统休眠（Hibernation）只能写入传统 swapfile，不影响 Zram 使用；唤醒后 Zram 重置，swapfile 中的休眠镜像被正常读取。

### 验证与检查

- 重启后运行 `swapon --show` 查看优先级，确认 Zram 优先（PRIO 列数值更高）。
- 若需调整，可重新编辑 fstab 或 zram-generator 配置，并重启或执行 `sudo swapon -a` 使更改生效。

```fstab
# ============================================================================
# Btrfs 子卷挂载（使用 noatime，调整压缩级别，添加 autodefrag）
# ============================================================================

# 根目录（系统）
UUID=fcb0aa3e-7abd-474d-9104-d82d9464697e /          btrfs rw,noatime,compress=zstd:1,ssd,discard=async,space_cache=v2,autodefrag,subvol=/@ 0 0

# /home（用户数据，压缩收益高）
UUID=fcb0aa3e-7abd-474d-9104-d82d9464697e /home      btrfs rw,noatime,compress=zstd:1,ssd,discard=async,space_cache=v2,autodefrag,subvol=/@home 0 0

# /root（root 用户家目录）
UUID=fcb0aa3e-7abd-474d-9104-d82d9464697e /root      btrfs rw,noatime,compress=zstd:1,ssd,discard=async,space_cache=v2,autodefrag,subvol=/@root 0 0

# /var/log（日志文件，压缩收益低且频繁写入，禁用压缩）
UUID=fcb0aa3e-7abd-474d-9104-d82d9464697e /var/log   btrfs rw,noatime,compress=no,ssd,discard=async,space_cache=v2,subvol=/@log 0 0

# /var/cache（缓存，禁用压缩）
UUID=fcb0aa3e-7abd-474d-9104-d82d9464697e /var/cache btrfs rw,noatime,compress=no,ssd,discard=async,space_cache=v2,subvol=/@cache 0 0

# /opt（可选软件，按需可保留压缩）
UUID=fcb0aa3e-7abd-474d-9104-d82d9464697e /opt       btrfs rw,noatime,compress=zstd:1,ssd,discard=async,space_cache=v2,autodefrag,subvol=/@opt 0 0

# /.snapshots（快照元数据，无需压缩）
UUID=fcb0aa3e-7abd-474d-9104-d82d9464697e /.snapshots btrfs rw,noatime,compress=no,ssd,discard=async,space_cache=v2,subvol=/@snapshots 0 0

# /var/lib/libvirt（虚拟机磁盘，建议禁用 CoW 而非压缩，此处挂载保留压缩选项但实际会由 chattr 覆盖）
UUID=fcb0aa3e-7abd-474d-9104-d82d9464697e /var/lib/libvirt btrfs rw,noatime,compress=no,ssd,discard=async,space_cache=v2,subvol=/@libvirt 0 0

# /var/lib/ollama（AI 模型，大文件，建议禁用压缩）
UUID=fcb0aa3e-7abd-474d-9104-d82d9464697e /var/lib/ollama btrfs rw,noatime,compress=no,ssd,discard=async,space_cache=v2,subvol=/@ollama 0 0

# /var/lib/docker（容器存储，禁用压缩）
UUID=fcb0aa3e-7abd-474d-9104-d82d9464697e /var/lib/docker btrfs rw,noatime,compress=no,ssd,discard=async,space_cache=v2,subvol=/@docker 0 0

# /var/lib/containers（Podman 等，禁用压缩）
UUID=fcb0aa3e-7abd-474d-9104-d82d9464697e /var/lib/containers btrfs rw,noatime,compress=no,ssd,discard=async,space_cache=v2,subvol=/@containers 0 0

# ============================================================================
# Swap 子卷挂载（@swap 独立挂载到 /swap）
# ============================================================================
UUID=fcb0aa3e-7abd-474d-9104-d82d9464697e /swap      btrfs rw,noatime,compress=no,ssd,discard=async,space_cache=v2,subvol=/@swap 0 0

# ============================================================================
# Swapfile 激活（低优先级，作为 Zram 的后备）
# ============================================================================
/swap/swapfile none swap sw,pri=10 0 0

# ============================================================================
# tmpfs（/tmp 内存文件系统）
# ============================================================================
tmpfs /tmp tmpfs rw,nodev,nosuid,noexec,size=4G 0 0
```

### 挂载选项优化建议

- **去除冗余选项**：`noatime` 已隐含 `nodiratime`，只需保留 `noatime`。
- **压缩级别调整**：将 `compress=zstd:3` 改为 `zstd:1` 或 `zstd:2` 以平衡 CPU 开销；对日志、缓存等频繁写入目录可设为 `compress=no`。
- **添加自动碎片整理**：对 `/` 和 `/home` 等目录可添加 `autodefrag` 减少小文件碎片。
- **其他推荐选项**：保留 `discard=async`、`space_cache=v2`；`ssd` 选项可保留或省略（内核自动检测）。
- **特定目录禁用 CoW**：对 Docker、Libvirt 等随机写入密集的目录，使用 `chattr +C` 禁用写时复制（需在数据写入前执行）。

### Swapfile 的创建与挂载

- **选择独立子卷**：用户决定创建顶层 `@swap` 子卷并挂载到 `/swap`，而非在 `@` 内部创建 `/var/swap`。操作需先挂载顶层卷（ID 5）创建 `@swap`，再在 `/etc/fstab` 中添加挂载项。
- **使用原生工具生成 swapfile**：推荐 `btrfs filesystem mkswapfile --size 4g --uuid clear /swap/swapfile`，该命令自动禁用 CoW 和校验和。
- **开机自动激活**：仍需在 `/etc/fstab` 中添加 `/swap/swapfile none swap defaults 0 0`（或 `sw,pri=10`）使系统启动时执行 `swapon`。

## chattr 命令与文件属性

### chattr 基本概念

- **命名**：`chattr` 即 "Change Attributes"，用于修改文件系统级别的扩展属性。
- **常用属性标志**：
  - `+i`（不可更改）：文件无法修改、删除或重命名。
  - `+a`（仅追加）：只能追加写入，不能覆盖或删除原有内容。
  - `+d`（不转储）：执行 `dump` 备份时跳过该文件。
  - `+C`（禁用 CoW，Btrfs 特有）：禁用写时复制。
- **语法**：`+` 添加、`-` 移除、`=` 设定为确切值。

### chattr 与 chmod 的区别

- **控制层面**：`chmod` 管理用户访问权限（读/写/执行），`chattr` 管理系统/内核处理文件的方式（如不可变、仅追加）。
- **作用范围**：`chmod` 针对用户、组、其他人分别设置；`chattr` 全局生效，root 用户也受限制。
- **跨平台性**：`chmod` 是标准 POSIX 权限，所有文件系统支持；`chattr` 属性为特定文件系统（如 Btrfs 的 `+C`）专有。

## NTFS 分区挂载与权限问题

### NTFS 挂载权限解决方案

- **问题根源**：内核 `ntfs3` 驱动默认权限较严格，且不识别 `umask` 选项（需使用 `fmask`/`dmask`）。
- **解决思路**：
  - 使用 `lowntfs-3g`（或 `ntfs-3g`）驱动并设置 `umask=000` 会强制所有用户可读写，但存在安全风险。
  - 更安全的方式：为 `ntfs3` 指定 `fmask=133,dmask=022`（文件 644、目录 755）或更宽松的 `fmask=000,dmask=000`。
- **查看 UID**：使用 `id -u` 查看当前用户 UID；`id -u 用户名` 查看指定用户；`/etc/passwd` 列出所有用户 UID。
- **新用户 UID**：系统从 1000 开始顺序分配（1000、1001、…），每个用户 UID 唯一。

# 另一组对话

> 用户计划在新笔记本（星曜14 AI9 365，32GB内存）上配置双硬盘（1TB主盘 + 2230系统盘），安装Windows + Arch Linux双系统，运行Colibri大模型项目，要求高性能、数据共享、Btrfs快照等特性。

## 硬盘与文件系统规划

### 2230系统盘选型与速度要求
- **接口规格**：星曜14 AI9 365的2230插槽为 **M.2 2230 物理尺寸，PCIe 4.0 x4 协议，M Key 接口**。
- **容量选择**：系统盘只需 **64-128GB**（推荐128GB），因系统本身占用不大。
- **速度建议**：顺序读取 **5000 MB/s 左右**为性价比甜点；实际流畅度更依赖 **4K 随机读写性能**。
- **选购要点**：优先 **TLC颗粒**，关注散热，2230规格通常无DRAM缓存（依赖HMB），高负载可能降速。

### 文件系统性能对比
- **XFS**：大文件与高并发场景性能最强，顺序读可达3.2GB/s。
- **EXT4**：稳定可靠，混合读写和小文件场景表现优异，是通用首选。
- **Btrfs**：功能丰富（快照、压缩、校验和），但随机写入性能约为EXT4的**60%**，需为功能付出性能代价。
- **F2FS**：专为闪存设计，特定随机写入场景有优势。
- **NTFS在Linux下**：`ntfs-3g`（FUSE）性能极差（4K随机写入仅EXT4的1/8）；`ntfs3`（内核驱动）可达EXT4的92%；新兴 `ntfs` 驱动性能更强但尚处实验阶段。
- **WSL2读取EXT4**：因绕过9P协议，由原生Linux内核直接管理，顺序读可达2.8GB/s，远优于商业驱动。

### 读写速度受文件系统影响
- 文件系统设计（元数据开销、写入机制、CPU效率）直接影响速度。
- Btrfs的CoW（写时复制）会带来额外写入开销；ZFS资源消耗极高，不推荐笔记本使用。
- 挂载参数（如 `noatime`、`nodatacow`、`compress`）对性能有显著影响。

## 分区与子卷方案设计

### 双硬盘分区布局（最终推荐）
**1TB主盘（Windows侧重）**：
- ESP（EFI）：**500MB-1GB**，FAT32（推荐1GB）
- Windows系统（C:）：**200-400GB**，NTFS
- Windows数据（D:）：**剩余大部分空间**，NTFS（存放文档、视频等低频数据，通过软链接共享给Linux）
- Data分区：**独立Ext4分区**（建议300-500GB），存放Colibri模型权重等高性能需求数据

**2230系统盘（Linux侧重）**：
- ESP（EFI）：**500MB-1GB**，FAT32（独立于Windows ESP，实现双系统物理隔离）
- Btrfs根分区：**剩余全部空间**，包含所有子卷

### Btrfs子卷布局建议
- `@` → 挂载到 `/`（根目录）
- `@home` → 挂载到 `/home`
- `@snapshots` → 挂载到 `/.snapshots`
- `@log` → 挂载到 `/var/log`
- `@cache` → 挂载到 `/var/cache`
- `@tmp` → 挂载到 `/tmp`（**走磁盘，不走tmpfs，以节省内存**）
- `@swap` → 挂载到 `/swap`（存放swapfile）
- `@containers`、`@docker`、`@ollama` 等 → 分别挂载到对应目录

### 关键设计决策说明
- **双EFI分区**：两个硬盘各建一个EFI分区，实现系统物理隔离，互不干扰。
- **/boot不单独分区**：作为Btrfs子卷（`@boot`），但需**单独禁用CoW**（`chattr +C`）以避免GRUB兼容性问题。
- **/tmp走磁盘子卷**：放弃tmpfs，避免挤占16GB内存空间，确保Colibri的25GB内存需求。
- **Data分区用Ext4**：模型权重文件放在独立Ext4分区，避免Btrfs的CoW和压缩带来额外开销。

## Swap与ZRAM配置策略

### Swap的作用与方案对比
- **Swap用途**：内存溢出保护 + 冷数据置换 + 系统休眠支持。
- **Swap分区**：性能稳定但大小固定，调整麻烦。
- **Swap文件**：灵活调整，现代内核性能与分区几乎无差，但Btrfs上需**禁用CoW**（`chattr +C`）。
- **ZRAM**：内存压缩交换，速度快（接近内存），但消耗CPU，占用物理内存作为存储池。
- **ZSWAP**：作为内存与磁盘Swap间的缓存层，热数据留内存，冷数据落盘。

### 针对32GB内存 + Colibri（需25GB）的推荐配置
- **ZRAM**：设置 **4-6GB**，优先级100（最高），作为高速缓存缓冲。
- **Swapfile**：设置 **16-20GB**，优先级-1（最低），作为最后防线。
- **总虚拟内存**：32GB物理 + 4GB ZRAM + 16GB Swap ≈ 52GB，足够应对峰值。
- **ZRAM空间特性**：**动态占用**，非静态预留，用多少占多少，实际物理占用远小于设定值。

### Btrfs上Swapfile的正确配置
```bash
# 在@swap子卷挂载后，确保子卷已禁用CoW
sudo chattr +C /swap
sudo dd if=/dev/zero of=/swap/swapfile bs=1M count=16384 status=progress
sudo chmod 600 /swap/swapfile
sudo mkswap /swap/swapfile
sudo swapon /swap/swapfile
```
- **验证**：`lsattr /swap/swapfile` 应显示 `C` 标志。

## 挂载参数优化

### 当前配置问题分析（用户现有配置）
- **物理内存不足**：仅14GB，远低于Colibri需求的25GB。
- **Swapfile CoW未关闭**：虽文件有`C`属性，但父卷挂载仍带`compress=zstd:2`，存在元数据开销。
- **autodefrag**：全局开启，对大文件顺序读取无益，徒增CPU和I/O负载。
- **nodiratime冗余**：`noatime`已包含`nodiratime`。

### 新机器推荐挂载参数矩阵
**系统核心子卷（`@`、`@home`、`@log`、`@cache`）**：
```fstab
UUID=xxx  /          btrfs  rw,noatime,compress=zstd:2,ssd,discard=async,space_cache=v2  0 0
```
（去掉 `autodefrag`）

**高负载/大文件子卷（`@containers`、`@docker`、`@ollama`）**：
```fstab
UUID=xxx  /var/lib/containers  btrfs  rw,noatime,nodatacow,ssd,discard=async,space_cache=v2  0 0
```
（关闭压缩和CoW）

**Swap专用子卷（`@swap`）**：
```fstab
UUID=xxx  /swap      btrfs  rw,noatime,nodatacow,noauto  0 0
```

**/tmp子卷**：
```fstab
UUID=xxx  /tmp       btrfs  rw,noatime,nodatacow,ssd,discard=async,space_cache=v2  0 0
```

## /tmp自动清理方案

### 使用systemd-tmpfiles管理（推荐）
创建 `/etc/tmpfiles.d/tmp.conf`：
```conf
# 清理/tmp下超过1天未被访问的文件
d /tmp 1777 root root 1d
```
- 默认 `systemd-tmpfiles-clean.timer` 每天触发一次清理。
- 可手动测试：`sudo systemd-tmpfiles --clean /etc/tmpfiles.d/tmp.conf`

### 备选方案（不推荐）
使用cron定时任务：`0 3 * * * /usr/bin/find /tmp -type f -atime +1 -delete`

## 跨系统数据共享方案

### NTFS D盘（文档、视频等低频数据）
- Linux通过 **软链接** 将 `~/Desktop`、`~/Documents` 等指向D盘对应文件夹。
- **挂载参数优化**：`fmask=133,dmask=022,noatime`。
- **性能代价**：读写NTFS在Linux下性能低于原生文件系统，但低频操作影响可忽略。

### Ext4 Data分区（模型权重等高性能数据）
- 独立分区，直接挂载到 `/mnt/data` 或 `/opt/models`。
- **Windows读取方案**：
  - **WSL2挂载**：性能最佳（顺序读2.8GB/s），通过 `wsl --mount` 命令。
  - **Paragon ExtFS**：商业软件，性能次之（顺序读2.1GB/s）。
  - **Ext2Read**：免费只读工具，速度慢（30-50MB/s），仅应急使用。
- **务必在Windows中禁用“快速启动”**，否则可能导致数据损坏。

## 启动引导方案

### GRUB能否引导另一硬盘的Windows
- **完全可行**：GRUB通过 `os-prober` 可扫描所有ESP分区中的Windows引导项。
- **推荐方案**：双硬盘各建独立ESP，BIOS设置Linux盘为第一启动项，实现物理隔离。

### /boot子卷与GRUB兼容性
- **问题根源**：Btrfs的压缩和扩展属性可能导致GRUB读取内核/initramfs失败。
- **解决方案**：
  - 为 `/boot` 创建独立子卷（如 `@boot`），**单独禁用CoW**（`chattr +C`）并禁用压缩。
  - 这样既保留在Btrfs快照内，又避免GRUB兼容问题。

### 替代启动加载器
- **Limine**：新兴启动器，原生支持Btrfs快照启动。
- **rEFInd + refind-btrfs**：图形化UEFI管理器，自动列出快照。
- **grub-btrfs**：GRUB插件，自动添加快照引导项。
- **systemd-boot**：不原生支持快照，社区方案不够稳定。

## 性能优化原则总结

1. **优先保证可用内存容量，而非追求内存速度**：OOM Kill是最大性能杀手。
2. **区分对待不同负载**：系统/日志保留压缩；模型/容器/swap关闭CoW和压缩。
3. **善用ZRAM + Swapfile双层策略**：ZRAM做高速缓冲，Swapfile做最后防线。
4. **避免全局优化陷阱**：`autodefrag`对大文件无益，应全局移除。
5. **硬件是根本**：32GB内存是运行Colibri的门槛，NVMe SSD顺序读5000MB/s以上为佳。

---

> 本总结涵盖从硬盘选型、分区规划、文件系统对比、挂载参数优化、Swap/ZRAM配置到跨系统共享的全部关键决策点，可直接作为新机器部署的操作指南。
