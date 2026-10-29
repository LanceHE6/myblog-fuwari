---
title:  Hysteria 2 代理节点极简部署与 Mihomo 自动容灾配置指南
description: 快速部署基于 QUIC/UDP 协议的高性能 Hysteria 2 代理节点，并配置客户端/网关实现“国外走代理、国内走直连、节点失联自动降级直连”的高可用分流。
category: Linux
tags:
- Clash
- 代理
published: 2026-10-29
---

# Hysteria 2 代理节点极简部署与 Mihomo 自动容灾配置指南

> **适用场景**：快速部署基于 QUIC/UDP 协议的高性能 Hysteria 2 代理节点，并配置客户端/网关实现“国外走代理、国内走直连、节点失联自动降级直连”的高可用分流。

---

## 一、服务端：Hysteria 2 快速部署

在 Linux VPS 终端中执行以下命令（自动完成官方安装、生成自签名 ECC 证书并配置启动）：

```bash
mkdir -p /etc/hysteria && \
curl -fsSL https://get.hy2.sh/ | bash && \
openssl req -x509 -nodes -newkey ec:<(openssl ecparam -name prime256v1) \
  -keyout /etc/hysteria/server.key \
  -out /etc/hysteria/server.crt \
  -subj "/CN=bing.com" -days 3650 && \

cat << 'EOF' > /etc/hysteria/config.yaml
listen: :45678

tls:
  cert: /etc/hysteria/server.crt
  key: /etc/hysteria/server.key

auth:
  type: password
  password: <YOUR_PASSWORD>

masquerade:
  type: proxy
  proxy:
    url: https://bing.com
    rewriteHost: true
EOF

chmod 644 /etc/hysteria/* && \
systemctl restart hysteria-server && \
systemctl enable hysteria-server
```

> **注意**：
>
> - 监听端口 `45678`（UDP）可根据需要自定义。
> - `<YOUR_PASSWORD>` 请替换为自定义强密码。
> - 确保云厂商安全组与防火墙放行该 UDP 端口。

---

## 二、服务验证

在 VPS 上检查服务运行状态与 UDP 端口监听：

```bash
systemctl status hysteria-server --no-pager
ss -ulnp | grep 45678
```

---

## 三、客户端节点配置

### 1. 节点直连 URL 格式

适用于支持直接导入 URL 的客户端（如 Sing-box、v2rayN、Shadowrocket 等）：

```text
hysteria2://<YOUR_PASSWORD>@<SERVER_IP>:45678/?sni=bing.com&insecure=1#My-Hy2-Node
```

### 2. Clash / Mihomo 原生配置语法

在配置文件的 `proxies:` 列表中添加：

```yaml
proxies:
  - name: "My-Hy2-Node"
    type: hysteria2
    server: "<SERVER_IP>"
    port: 45678
    password: "<YOUR_PASSWORD>"
    sni: bing.com
    skip-cert-verify: true
```

---

## 四、Mihomo / TUN 模式高可用与全自动容灾配置

适用于家庭服务器、软路由或 Linux 机器部署的 Mihomo 透明代理配置。实现：

- **国外流量**：走 Hysteria 2 节点。
- **国内流量**：直连。
- **高可用保底**：当海外节点出现网络抖动或失联时，`fallback` 策略组在 30 秒内自动且无感降级为“直连”，防止因节点故障引发整机网络瘫痪。

完整 `config.yaml` 模板：

```yaml
mixed-port: 7890
allow-lan: true
mode: rule
log-level: info
ipv6: false
unified-delay: true
tcp-concurrent: true

profile:
  store-selected: true

external-controller: '0.0.0.0:9096'
secret: "password"

# --- 系统级 TUN 模式 ---
tun:
  enable: true
  stack: system
  device: mihomo-tun
  mtu: 1500
  gso: true
  gso-max-size: 65536
  dns-hijack:
    - "any:53"
    - "tcp://any:53"
  auto-route: true
  auto-redirect: true
  auto-detect-interface: true

# --- 节点定义 ---
proxies:
  - name: "My-Hy2-Node"
    type: hysteria2
    server: "<SERVER_IP>"
    port: 45678
    password: "<YOUR_PASSWORD>"
    sni: bing.com
    skip-cert-verify: true

  - name: "直连"
    type: direct
    udp: true

# --- DNS 配置 ---
dns:
  enable: true
  ipv6: false
  respect-rules: true
  enhanced-mode: fake-ip
  nameserver:
    - https://223.5.5.5/dns-query
    - https://120.53.53.53/dns-query
  nameserver-policy:
    "geosite:cn,private":
      - https://223.5.5.5/dns-query
      - https://120.53.53.53/dns-query
    "geosite:geolocation-!cn":
      - "https://dns.cloudflare.com/dns-query"
      - "https://dns.google/dns-query"

# --- 策略组（核心：Fallback 自动容灾降级） ---
proxy-groups:
  # 优先走代理节点；若节点失联，自动降级为“直连”
  - name: 默认
    type: fallback
    proxies:
      - My-Hy2-Node
      - 直连
    url: "https://cp.cloudflare.com/generate_204"
    interval: 30

  - name: 国内
    type: select
    proxies:
      - 直连
      - 默认

  - name: 国外
    type: select
    proxies:
      - 默认
      - My-Hy2-Node
      - 直连

# --- 分流规则 ---
rules:
  # 局域网与私有地址直连
  - GEOIP,lan,直连,no-resolve
  - GEOSITE,private,直连

  # 国内服务与国内 IP 直连
  - GEOSITE,CN,国内
  - GEOIP,CN,国内

  # 国外流量走代理（失联时自动 fallback 到直连）
  - GEOSITE,geolocation-!cn,国外
  - MATCH,国外
```

---
