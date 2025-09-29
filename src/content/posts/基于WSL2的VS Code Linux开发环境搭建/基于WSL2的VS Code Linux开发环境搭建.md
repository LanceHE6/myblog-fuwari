---
title: 基于WSL2的VS Code Linux开发环境搭建
description: 使用Ubuntu WSL2 + VS Code搭建Linux开发环境
# image: ./cover.png
category: Linux
tags:
- wsl
published: 2025-09-30

draft: true
---


## wsl无法访问github

sudo rm /etc/resolv.conf
sudo bash -c 'echo "nameserver 8.8.8.8" > /etc/resolv.conf'
sudo bash -c 'echo "[network]" > /etc/wsl.conf'
sudo bash -c 'echo "generateResolvConf = false" >> /etc/wsl.conf'
