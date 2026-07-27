---
title: 部署并配置 Xray (VLESS + REALITY) 抗封锁代理
description: 介绍如何从零部署当前抗封锁能力最强、稳定性最高的Xray (VLESS + REALITY)架构，实现无域名绑定的高质量网络连接
category: Linux
tags:
- Clash
- 代理
published: 2026-07-27
---

# 部署并配置 Xray (VLESS + REALITY) 抗封锁代理

> **文章摘要**：传统的加密代理协议（如 Shadowsocks、旧版 VMess）随着防火墙 DPI（深度包检测）和主动探测技术的演进，阻断率日益升高。本文将详细介绍如何从零部署当前抗封锁能力最强、稳定性最高的 **Xray (VLESS + REALITY)** 架构，实现无域名绑定的高质量网络连接。

---

## 为什么选择 VLESS + REALITY？

在选择科学上网架构时，**稳定性**和**抗封锁能力**是核心指标。相比传统方案，`VLESS + REALITY` 具备以下杀手级优势：

1. **无需自备域名**：直接借用公网大厂（如 Apple、Microsoft、Google）的合规 TLS 证书与 SNI，省去域名购买与续费成本。
2. **免疫主动探测**：当防火墙发送伪造的探测包时，REALITY 会将请求反向代理至真正的目标网站（如 Apple 官网），返回合法的响应，从而彻底骗过 DPI 检测。
3. **前沿加密与流控**：结合 `xtls-rprx-vision` 流控算法，去除明显的 TLS in TLS 特征，传输效率极高。

---

## 准备工作

* **服务器**：一台干净的 VPS（推荐操作系统：Ubuntu 20.04 / 22.04 / 24.04 或 Debian 11 / 12）。
* **客户端**：支持 REALITY 协议的客户端（如 v2rayN, Clash Verge Rev, Sing-box, NekoBox 等）。
* **终端工具**：SSH 连接工具（如 Termius, PuTTY 或 本地 Terminal）。

---

## 第一步：环境准备与一键安装 Xray

登录 VPS 终端（建议使用 `root` 用户），执行以下命令更新系统依赖并调用 Xray 官方一键脚本：

```bash
# 1. 更新系统软件包
apt update && apt upgrade -y

# 2. 安装基础依赖工具
apt install -y curl wget tar jq openssl

# 3. 执行 Xray 官方安装脚本
bash <(curl -L [https://github.com/XTLS/Xray-install/raw/main/install-release.sh](https://github.com/XTLS/Xray-install/raw/main/install-release.sh))
```

安装成功后，Xray 会被注册为系统服务，主配置文件路径为 `/usr/local/etc/xray/config.json`。

---

## 第二步：生成核心密钥与参数

VLESS + REALITY 的运行依赖 **UUID**、**X25519 密钥对** 以及 **Short ID**。在终端中依次运行以下命令生成参数并妥善记录：

### 1. 生成用户 UUID
```bash
xray uuid
```
> **示例输出**：`a1b2c3d4-e5f6-7890-abcd-1234567890ab`

### 2. 生成 REALITY 密钥对
```bash
xray x25519
```
> **示例输出**：
> * `Private key`: `uX8...（服务端使用）`
> * `Public key`: `9jK...（客户端使用）`

### 3. 生成 Short ID
```bash
openssl rand -hex 8
```
> **示例输出**：`1a2b3c4d5e6f7a8b`

> ⚠️ **关键注意**：
> * **Private Key（私钥）**：仅保存在服务端配置文件中，切勿泄露！
> * **Public Key（公钥）**：用于客户端连接配置。

---

## 第三步：修改服务端配置文件

编辑配置文件：

```bash
vim /usr/local/etc/xray/config.json
```

清空原内容，替换为以下完整配置（将对应位置替换为你刚才生成的真实参数）：

```json
{
  "log": {
    "loglevel": "warning"
  },
  "inbounds": [
    {
      "port": 443,
      "protocol": "vless",
      "settings": {
        "clients": [
          {
            "id": "你的UUID",
            "flow": "xtls-rprx-vision"
          }
        ],
        "decryption": "none"
      },
      "streamSettings": {
        "network": "tcp",
        "security": "reality",
        "realitySettings": {
          "show": false,
          "dest": "[www.apple.com:443](https://www.apple.com:443)",
          "xver": 0,
          "serverNames": [
            "[www.apple.com](https://www.apple.com)"
          ],
          "privateKey": "你的Private_Key",
          "shortIds": [
            "你的Short_ID"
          ]
        }
      }
    }
  ],
  "outbounds": [
    {
      "protocol": "freedom"
    }
  ]
}
```

---

## 第四步：校验配置并启动服务

1. **测试配置文件语法**：
   ```bash
   xray run -test -config /usr/local/etc/xray/config.json
   ```
   *若返回 `Configuration OK`，说明 JSON 格式完全正确。*

2. **启动 Xray 并设置开机自启**：
   ```bash
   systemctl restart xray
   systemctl enable xray
   ```

3. **检查服务运行状态**：
   ```bash
   systemctl status xray --no-pager
   ```
   *确认状态显示为绿色的 `active (running)`。*

---

## 第五步：防火墙与安全组设置

REALITY 默认使用标准 HTTPS 端口 `443`。确保系统防火墙和云厂商控制台的安全组已开放该端口：

```bash
# 若系统启用了 UFW 防火墙
ufw allow 443/tcp
```

---

## 第六步：客户端对接参数对照表

服务配置完成后，在客户端新增 VLESS 节点，参数映射如下：

| 配置项 | 填写说明 / 推荐值 | 备注 |
| :--- | :--- | :--- |
| **服务器地址 (Address)** | 你的 VPS 公网 IP | 例如：`156.233.x.x` |
| **端口 (Port)** | `443` | 与服务端监听端口一致 |
| **协议 (Protocol)** | `VLESS` | — |
| **用户 ID (UUID)** | 生成的 `UUID` | — |
| **传输协议 (Network)** | `tcp` | — |
| **流控 (Flow)** | `xtls-rprx-vision` | 核心优化项，建议必填 |
| **传输层安全 (TLS)** | `REALITY` | — |
| **伪装域名 (SNI)** | `www.apple.com` | 必须与服务端 `serverNames` 完全一致 |
| **客户端指纹 (Fingerprint)**| `chrome` | 模拟常用浏览器 TLS 指纹 |
| **公钥 (Public Key)** | 生成的 **`Public Key`** | ⚠️ 千万不要填成 Private Key |
| **简短 ID (Short ID)** | 生成的 `Short ID` | — |


如果你使用的是 **Clash Verge**，可新建一个本地 Profile（`.yaml` 文件），直接复制以下经过格式修正的模版：

```yaml
# 1. 节点定义 (Proxies)
proxies:
  - name: "REALITY-节点"
    type: vless
    server: 你的VPS公网IP            # 替换为你的服务器 IP
    port: 443                        # 服务端端口
    uuid: 你的UUID                    # 替换为你生成的 UUID
    udp: true
    tls: true
    flow: xtls-rprx-vision
    servername: [www.apple.com](https://www.apple.com)        # 伪装域名 (SNI)
    network: tcp
    reality-opts:
      public-key: 你的PublicKey       # 替换为 Public Key (公钥)
      short-id: 你的ShortID          # 替换为 Short ID
    client-fingerprint: chrome       # 客户端 TLS 指纹

# 2. 策略组 (Proxy Groups)
proxy-groups:
  - name: 节点选择
    type: select
    proxies:
      - "REALITY-节点"
      - DIRECT

  - name: 漏网之鱼
    type: select
    proxies:
      - 节点选择
      - DIRECT

# 3. 基础分流规则 (Rules)
rules:
  - GEOIP,lan,DIRECT
  - GEOIP,cn,DIRECT
  - MATCH,漏网之鱼
```

---

## 💡 维护与常见问题（FAQ）

1. **连接提示 Timeout 或超时？**
    * 排查 VPS 云控制台（如阿里云、AWS、甲骨文等）安全组是否放行了 `TCP 443` 端口。
    * 排查本地网络是否存在防火墙拦截。

2. **伪装域名如何选择？**
    * 优先选择支持 **TLS 1.3 / HTTP2** 且服务器地理位置相近的大厂域名（如 `www.apple.com`、`dl.google.com`、`gateway.icloud.com`）。

3. **服务重启或查看日志命令**：
    * 查看实时日志：`journalctl -u xray -f`
    * 重启服务：`systemctl restart xray`
```