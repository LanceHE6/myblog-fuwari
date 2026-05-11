---
title: Windows更新导致双系统GRUB引导损坏修复指南
description: 记录因Windows系统更新导致GRUB无法识别原有文件系统而无法正常进入系统的解决办法
#image: /cover/cover2.png
category: GRUB
tags:
- 双系统
  #sticky: 999
published: 2026-05-11
---

# Windows更新导致双系统GRUB引导损坏修复指南

### 0. 问题背景

笔记本采用 **Windows 11 + Manjaro (Linux)** 的双系统组合。Windows 系统更新（尤其是大版本更新）经常会篡改 UEFI 启动项或导致 GRUB 无法识别原有的文件系统，导致系统重启后瘫痪。

### 1. 故障现象

重启电脑后，没有出现熟悉的系统选择菜单，而是直接进入了黑底白字的命令行界面，报错信息如下：

```bash
error: kern/fs.c:grub_fs_probe:122:unknown filesystem.
Entering rescue mode...
grub rescue> _

```

这意味着 GRUB 的核心文件虽然还在，但它无法识别或找到存放配置文件（grub.cfg）的分区，导致引导中断。

---

### 2. 核心修复步骤（Live USB 模式）

最稳妥的方法是使用 **Manjaro Live USB** 启动盘。

#### 步骤 A：定位 btrfs 子卷

由于 Manjaro 的 btrfs 布局通常将系统安装在名为 `@` 的子卷中，普通的 `mount` 命令无法让 `chroot` 工具识别系统环境。

1. **识别分区**：使用 `lsblk -f` 确认分区。
* 根分区（btrfs）：例如 `/dev/nvme1n1p7`
* EFI 分区（vfat）：例如 `/dev/nvme1n1p5`

2. **手动挂载子卷**：
```bash
# 必须指定 subvol=@ 才能进入真正的根目录
sudo mount -o subvol=@ /dev/nvme1n1p7 /mnt
# 挂载 EFI 分区
sudo mount /dev/nvme1n1p5 /mnt/boot/efi

```

#### 步骤 B：进入 Chroot 环境修复

1. **进入环境**：
```bash
sudo manjaro-chroot /mnt /bin/bash

```

2. **重装 GRUB**：
```bash
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=manjaro --recheck

```

3. **更新配置**：
```bash
update-grub

```


*注意：如果此时没看到 Windows 引导项，请先重启进入 Manjaro 桌面后再执行一次 `sudo update-grub`。*

---

### 3. 为什么会发生这种情况？

1. **启动项篡改**：Windows 更新会自我膨胀，强行将 `Windows Boot Manager` 设为 UEFI 的第一优先级。
2. **文件系统锁定**：Windows 的“快速启动”本质是休眠，会导致磁盘处于“脏（Dirty）”状态，GRUB 有时因此无法读取分区。
3. **分区表变动**：更新可能导致分区 UUID 或顺序微调，让原本硬编码在引导程序里的路径失效。

---

### 4. 预防与杜绝措施

#### 1. 关闭 Windows 快速启动

进入 **控制面板 > 电源选项 > 选择电源按钮的功能**，点击“更改当前不可用的设置”，**取消勾选“启用快速启动”**。这能确保 Windows 彻底释放硬盘权限。

#### 2. 在 Windows 中锁定引导顺序

以管理员权限打开 PowerShell，执行：

```powershell
bcdedit /set {bootmgr} path \EFI\manjaro\grubx64.efi

```

这告诉 Windows 的引导管理器：即便你想当老大，也要通过 Manjaro 的 GRUB 进门。

#### 3. BIOS 设置优化

* **关闭 Secure Boot**：减少由于签名校验导致的引导失败。
* **调整 Boot Priority**：确保 BIOS 内 `manjaro` 始终排在第一位。
