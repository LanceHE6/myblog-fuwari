---
title: Ubuntu设置允许root用户ssh登录
description: Ubuntu设置允许root用户ssh登录
#cover: /cover/cover2.png
tags:
 - Linux
 - ssh
#sticky: 999
published: 2025-09-19
---

* 1.设置root用户密码
```shell
sudo passwd root
```
* 2.修改sshd_config文件
```shell
sudo vim /etc/ssh/sshd_config
```
修改下面配置
```text
#PermitRootLogin prohibit-password
#PasswordAuthentication yes
```
为
```text
PermitRootLogin yes
PasswordAuthentication yes
```
* 3.重启ssh服务
```shell
sudo service sshd restart
```