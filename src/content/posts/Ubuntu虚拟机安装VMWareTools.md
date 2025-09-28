---
title: Ubuntu虚拟机安装VMWare Tools
description: Ubuntu虚拟机安装VMWare Tools
#image: /cover/cover2.png
category: Linux
tags: [Ubuntu, 虚拟机]
published: 2025-01-31
---

# Ubuntu虚拟机安装VMWare Tools

在大多数情况下VMWare会自动给Ubuntu虚拟机安装VMWare tools,但难免会存在系统版本太新,没有网络等其他导致VMWare tools没法安装的情况

没有VMWare tools,可能会导致显示的屏幕大小无法适配,无法在宿主机和虚拟机之间复制粘贴文件,文本等

## 解决方案

### 一.更新软件包

```shell
sudo apt update
```

### 二.卸载旧版本的VMWare tools

```shell
sudo apt autoremove open-vm-tools
```

### 三.安装新版本的VMWare tools

```shell
sudo apt install open-vm-tools-desktop
```

### 四.重启系统


参考文章: https://blog.csdn.net/ylong52/article/details/140824569
