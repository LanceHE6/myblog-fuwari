---
title: Manjaro安装Clash Verge Rev并配置TUN模式代理
description: 在Manjaro系统上安装Clash Verge Rev，配置Mihomo内核并启用TUN模式实现全局透明代理
category: Linux
tags:
- Clash
- 代理
- Manjaro
published: 2026-06-30
---

# Manjaro安装Clash Verge Rev并配置TUN模式代理

## 背景

Clash Verge Rev 是 Clash Verge 的社区延续版本，基于 Tauri 构建，使用 Mihomo（原 Clash Meta）作为代理内核。相比原版，Clash Verge Rev 持续更新维护，功能更加完善。

TUN 模式可以在系统网络层创建一个虚拟网卡，将所有流量（包括非代理应用的流量）路由到代理内核，实现真正的全局透明代理。这对于终端代理、虚拟机网络等场景非常实用。

## 环境

* 系统：Manjaro（Arch 系通用）
* 桌面环境：KDE Plasma（其他桌面环境操作类似）

## 安装 Clash Verge Rev

### 通过 AUR 安装

Manjaro 默认已配置好 AUR，使用 `yay` 安装预编译版本即可（推荐使用 `-bin` 版本，无需本地编译）：

```shell
yay -S clash-verge-rev-bin
```

安装过程中提示选择时使用默认选项即可。安装完成后即可在应用菜单中找到 Clash Verge Rev。

### 使用 pacman 安装（如果可用）

部分 Manjaro 源或社区源也可能收录了此包：

```shell
sudo pacman -S clash-verge-rev-bin
```

### 备用方式：AppImage

如果 AUR 安装遇到问题，也可以从 [GitHub Releases](https://github.com/clash-verge-rev/clash-verge-rev/releases) 下载 AppImage 版本直接运行。

## 安装 Mihomo 内核

Clash Verge Rev 默认不携带代理内核，需要单独安装 Mihomo 内核。

### 通过 AUR 安装 Mihomo

```shell
yay -S mihomo
```

### 在 Clash Verge Rev 中指定内核路径

1. 打开 Clash Verge Rev
2. 进入 **设置** → **Clash 设置**
3. 在 **Mihomo 内核** 中填入 `/usr/bin/mihomo`
   ![alt text](image.png)

## 配置代理订阅

### 添加订阅

1. 打开 Clash Verge Rev
2. 进入 **配置** 页面
3. 点击 **新建**，选择 **导入订阅链接**
4. 填入订阅 URL 并保存
5. 右键订阅 → **更新**，拉取节点信息

### 选择节点

更新完成后，进入 **代理** 页面：
* 点击 **系统代理**，选择需要的节点
* 也可使用 **规则** 模式，根据规则自动分流

### 开启系统代理

返回主界面，点击右上角的开关启动代理。默认会在系统设置中自动配置 HTTP/HTTPS 代理（127.0.0.1:7890），浏览器即可通过代理访问。

## 配置 TUN 模式

TUN 模式通过创建虚拟网卡实现全局透明代理，让所有应用的流量都经过代理。

### 开启 TUN 模式

1. 在 Clash Verge Rev 主界面，找到 **TUN 模式** 开关
2. 点击开启
   ![alt text](image-1.png)

首次开启时，程序会弹出授权提示，要求输入密码以创建虚拟网卡。

### 授予管理员权限（如需要）

如果 TUN 模式无法正常启动，可能是因为 Mihomo 内核没有足够的权限。解决方法：

**方法一：使用 setcap 授予网络权限**

```shell
sudo setcap cap_net_admin=+ep /usr/bin/mihomo
```

**方法二：以 root 权限运行 Mihomo 内核**

在 Clash Verge Rev 设置中，可以配置以上下文用户权限运行内核，或者直接赋予 `/usr/bin/mihomo` 可执行权限和网络管理能力。

### TUN 模式配置参数

进入 **设置** → **Clash 设置** → **TUN 模式设置**，关键参数说明：

| 参数 | 建议值 | 说明 |
|---|---|---|
| 设备名 | Mihomo | 虚拟网卡名称 |
| IPv4 地址 | 198.18.0.1/30 | TUN 网卡的 IPv4 地址段 |
| IPv6 地址 | 关闭 | 一般无需 IPv6 |
| MTU | 9000 | 最大传输单元 |
| 自动路由 | 开启 | 自动配置路由表 |
| DNS 劫持 | 开启 | 劫持 DNS 请求防止泄漏 |

### 验证 TUN 模式是否生效

开启 TUN 模式后，通过以下方式验证：

1. 查看虚拟网卡是否创建成功：

```shell
ip addr show tun0
```

出现 `tun0` 网卡且 IP 为 `198.18.0.1` 即表示虚拟网卡创建成功。

2. 检查路由表是否已正确配置：

```shell
ip route show
```

3. 在终端中测试访问：

```shell
curl -I https://www.google.com
```

如果能正常返回，说明 TUN 模式已生效，终端流量也走了代理。

4. 在 Clash Verge Rev 的 **日志** 页面中也可以看到实时连接记录，可以确认流量经过代理。

## 设置开机自启

为了让系统开机后自动开启代理，可以配置 Clash Verge Rev 开机自启：

1. 打开 Clash Verge Rev
2. 进入 **设置** → **系统设置**
3. 开启 **开机自启**

## 常见问题

### TUN 模式启用后上不了网

* 检查 TUN 模式配置中的 IPv4 地址段是否与本机网络冲突，避免使用常见的 `192.168.x.x`、`10.x.x.x` 等地址段
* 检查 `/etc/resolv.conf` 是否被正确劫持到 `198.18.0.1`

### TUN 模式切换失败 / 状态卡住

* 可能是 Mihomo 内核进程未被正确关闭，手动结束残留进程：

```shell
pkill mihomo
```

然后重新在 Clash Verge Rev 中开启 TUN 模式。

### 某些应用不走代理

* TUN 模式下仍走规则分流，检查 **代理** 页面中的分流规则是否正确
* 可临时切换到 **全局** 模式测试，确认是否为规则问题

### 还原系统代理设置

若 Clash Verge Rev 异常退出导致系统代理设置未还原，可手动关闭：

```shell
# Gnome / GTK 环境
gsettings set org.gnome.system.proxy mode 'none'

# KDE 环境：进入 系统设置 → 网络 → 代理 → 关闭
```

## 参考

* [Clash Verge Rev GitHub](https://github.com/clash-verge-rev/clash-verge-rev)
* [Mihomo 文档](https://wiki.metacubex.one/)
