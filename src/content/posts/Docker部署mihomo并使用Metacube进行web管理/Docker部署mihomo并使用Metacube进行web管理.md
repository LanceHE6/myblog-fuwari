---
title: Docker部署mihomo并使用Metacube进行web管理
description: Docker部署mihomo和Metacube，并以TUN模式代理，实现在无桌面环境的服务器上透明代理
#image: /cover/cover2.png
category: Proxy
tags:
- docker
  #sticky: 999
published: 2026-05-08
---

# Docker部署mihomo并使用Metacube进行web管理

## Tun模式和普通代理模式的区别

1.普通代理模式(HTTP/Socket)下，代理程序会在系统中开启一个端口（默认为7890），**其他程序主动将流量发送至该端口**，但很多程序（如 ping、git）不读取系统代理设置，且不支持除TCP/UDP外的协议。属于应用层的代理。

2.Tun模式(Network TUNnel)，代理程序会在系统中生成一张名为 `utun` 或 `meta` 的虚拟网卡，并且会修改路由表，让所有流量都经过该网卡。属于网络层的代理。

| 特性       | 普通代理模式 (HTTP/Socks5) | TUN 模式 (全局代理)        |
|----------|----------------------|----------------------|
| 层级       | 应用层 (Layer 7)        | 网络层 (Layer 3)        |
| 通用性      | 差，需软件支持设置代理          | 极强，接管所有软件            |
| UDP/ICMP | 支持,较弱，取决于软件协议        | 强，支持 Ping 和各种 UDP 流量 |
| 主要用途     | 浏览器翻墙、特定程序加速         | 服务器全局加速、游戏加速、透明网关    |

## 准备工作

* Docker：确保环境已安装Docker和Docker compose
* 节点订阅：拥有可用的机场节点订阅链接

## 一、准备mihomo配置文件

1.在宿主机上创建一个文件夹用于存放mihomo的配置文件，例如`/opt/mihomo`

2.在该路径下创建`config.yaml`文件

3.参考以下模板填写配置文件内容

```yaml
# 基础配置
mixed-port: 7890
allow-lan: true
bind-address: '*'
mode: rule
log-level: info
external-controller: '0.0.0.0:9090' # 监听地址和端口
secret: "<API访问密钥>" # 用于Metacube登录时使用

tun:
  enable: true  # 使用tun模式
  stack: system
  dns-hijack:
    - "any:53"
    - "tcp://any:53"
  auto-route: true
  auto-redirect: true
  auto-detect-interface: true

proxy-providers:
  <节点提供者名称>:
    url: "<节点订阅链接>"
    type: http
    interval: 86400
    health-check: {enable: true,url: "https://www.gstatic.com/generate_204", interval: 300}
    override:
      additional-prefix: "<节点名称前缀>"

# 节点配置，策略组和规则。可直接使用订阅文件中的配置
proxies:
  - name: "直连"
    type: direct
    udp: true

geodata-mode: true
geox-url:
  geoip: "https://github.com/MetaCubeX/meta-rules-dat/releases/download/latest/geoip.dat"
  geosite: "https://github.com/MetaCubeX/meta-rules-dat/releases/download/latest/geosite.dat"
  mmdb: "https://github.com/MetaCubeX/meta-rules-dat/releases/download/latest/country-lite.mmdb"
  asn: "https://github.com/MetaCubeX/meta-rules-dat/releases/download/latest/GeoLite2-ASN.mmdb"

dns:
  enable: true
  ipv6: true
  respect-rules: true
  enhanced-mode: fake-ip
  fake-ip-filter:
    - "*"
    - "+.lan"
    - "+.local"
    - "+.market.xiaomi.com"
  nameserver:
    - https://120.53.53.53/dns-query
    - https://223.5.5.5/dns-query
  proxy-server-nameserver:
    - https://120.53.53.53/dns-query
    - https://223.5.5.5/dns-query
  nameserver-policy:
    "geosite:cn,private":
      - https://120.53.53.53/dns-query
      - https://223.5.5.5/dns-query
    "geosite:geolocation-!cn":
      - "https://dns.cloudflare.com/dns-query"
      - "https://dns.google/dns-query"

proxy-groups:

  - name: 默认
    type: select
    proxies: [自动选择,直连,香港,台湾,日本,新加坡,美国,其它地区,全部节点]

  - name: Google
    type: select
    proxies: [默认,香港,台湾,日本,新加坡,美国,其它地区,全部节点,自动选择,直连]

  - name: Telegram
    type: select
    proxies: [默认,香港,台湾,日本,新加坡,美国,其它地区,全部节点,自动选择,直连]

  - name: Twitter
    type: select
    proxies: [默认,香港,台湾,日本,新加坡,美国,其它地区,全部节点,自动选择,直连]

  - name: 哔哩哔哩
    type: select
    proxies: [默认,香港,台湾,日本,新加坡,美国,其它地区,全部节点,自动选择,直连]

  - name: 巴哈姆特
    type: select
    proxies: [默认,香港,台湾,日本,新加坡,美国,其它地区,全部节点,自动选择,直连]

  - name: YouTube
    type: select
    proxies: [默认,香港,台湾,日本,新加坡,美国,其它地区,全部节点,自动选择,直连]

  - name: 海外AI
    type: select
    proxies: [默认,香港,台湾,日本,新加坡,美国,其它地区,全部节点,自动选择,直连]

  - name: NETFLIX
    type: select
    proxies: [默认,香港,台湾,日本,新加坡,美国,其它地区,全部节点,自动选择,直连]

  - name: Spotify
    type: select
    proxies:  [默认,香港,台湾,日本,新加坡,美国,其它地区,全部节点,自动选择,直连]

  - name: Github
    type: select
    proxies:  [默认,香港,台湾,日本,新加坡,美国,其它地区,全部节点,自动选择,直连]

  - name: 国内
    type: select
    proxies:  [直连,默认,香港,台湾,日本,新加坡,美国,其它地区,全部节点,自动选择]

  - name: 其他
    type: select
    proxies:  [默认,香港,台湾,日本,新加坡,美国,其它地区,全部节点,自动选择,直连]

  #分隔,下面是地区分组
  - name: 香港
    type: select
    include-all: true
    filter: "(?i)港|hk|hongkong|hong kong"

  - name: 台湾
    type: select
    include-all: true
    filter: "(?i)台|tw|taiwan"

  - name: 日本
    type: select
    include-all: true
    filter: "(?i)日|jp|japan"

  - name: 美国
    type: select
    include-all: true
    filter: "(?i)美|us|unitedstates|united states"

  - name: 新加坡
    type: select
    include-all: true
    filter: "(?i)(新|sg|singapore)"

  - name: 其它地区
    type: select
    include-all: true
    filter: "(?i)^(?!.*(?:🇭🇰|🇯🇵|🇺🇸|🇸🇬|🇨🇳|港|hk|hongkong|台|tw|taiwan|日|jp|japan|新|sg|singapore|美|us|unitedstates)).*"

  - name: 全部节点
    type: select
    include-all: true

  - name: 自动选择
    type: url-test
    include-all: true
    tolerance: 10

rules:
  - GEOIP,lan,直连,no-resolve
  - GEOSITE,twitter,Twitter
  - GEOSITE,youtube,YouTube
  - GEOSITE,category-ai-!cn,海外AI
  - GEOSITE,github,Github
  - GEOSITE,google,Google
  - GEOSITE,telegram,Telegram
  - GEOSITE,netflix,NETFLIX
  - GEOSITE,bilibili,哔哩哔哩
  - GEOSITE,bahamut,巴哈姆特
  - GEOSITE,spotify,Spotify
  - GEOSITE,CN,国内
  - GEOSITE,geolocation-!cn,其他

  - GEOIP,CN,国内
  - MATCH,其他
```

## 二、使用DockerCompose部署

通过docker compose同时部署mihomo核心和metacube

1.在任意位置创建`docker-compose.yml`文件

2.根据实际环境修改文件

```yaml
version: '3'

services:
  # Web 管理面板
  metacubexd:
    container_name: metacubexd
    #image: ghcr.io/metacubex/metacubexd
    # 国内加速源
    image: ghcr.1ms.run/metacubex/metacubexd
    restart: always
    ports:
      - '9097:80' # WebUI 访问端口

  # Mihomo 核心
  mihomo:
    container_name: mihomo
    #image: docker.io/metacubex/mihomo:latest
    # 国内加速源
    image: docker.1ms.run/metacubex/mihomo:latest
    restart: always
    pid: host
    ipc: host
    network_mode: host # 推荐使用 Host 模式，性能最好且便于处理 IPv6
    
    # --- TUN 模式核心配置 ---
    cap_add:
      - NET_ADMIN      # 赋予网络管理权限
    devices:
      - /dev/net/tun:/dev/net/tun # 映射宿主机的 TUN 设备
    volumes:
      # ⚠️ 注意：冒号左边请修改为你 config.yaml 所在的实际绝对路径
      - /opt/mihomo:/root/.config/mihomo
```

3.在`docker-compose.yml`文件所在目录，使用`sudo docker-compose up -d`启动容器

## 三、登录WebUI

1.局域网访问`http://设备IP:9097`

2.后端地址`http://设备IP:9096`（端口以实际配置文件为准）
