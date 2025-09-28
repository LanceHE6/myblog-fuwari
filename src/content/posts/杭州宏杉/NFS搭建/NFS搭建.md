---
title: nfs搭建
description: 杭州宏杉学习笔记-nfs搭建
#cover: /cover/cover2.png
category: 杭州宏杉科技
tags:
- nfs
  #sticky: 999
published: 2025-09-19
---

## CentOs

### nfs服务端

#### 1.安装nfs

```shell
yum install -y nfs-utils rpcbind
```

* 查看是否安装成功：
```shell
rpm -qa nfs-utils
```
* 查看rpc服务是否自动启动
```shell
ss -tuln | grep 111
```

#### 2.设置开机启动nfs相关服务

```shell 
systemctl enable rpcbind
systemctl enable nfs-server
systemctl enable nfs-idmap
systemctl enable nfs-lock
```
#### 3.启动nfs服务

```shell
systemctl start rpcbind
systemctl start nfs-server
systemctl start nfs-idmap
systemctl start nfs-lock
```
#### 4.创建共享目录并修改权限
```shell
mkdir /nfs
chmod 777 /nfs
```
#### 5.修改配置文件
```shell
echo "/nfs *(rw,sync,no_root_squash)" >> /etc/exports
```

## Ubuntu

### nfs服务端

* 1.安装nfs服务器
```shell
sudo apt install nfs-kernel-server
```

* 2.创建共享目录

````shell
sudo mkdir /nfs
````

* 3.修改nfs服务器配置文件
```shell
sudo sudo /etc/exports
```
指定nfs服务器共享目录及其属性：
```shell
/nfs *(rw,sync,no_root_squash)

# * 允许所有网段访问，也可以使用具体IP
# rw 读写权力
# sync 数据同步写入内存和硬盘
# no_root_squash 允许root用户访问
# no_subtree_check 不检查父目录的权限
```
* 4.重启nfs服务
```shell
sudo systemctl restart nfs-kernel-server
# 或者
sudo service nfs-kernel-server restart
```
* 5.查看nfs服务器的共享目录
```shell
showmount -e localhost
```

### nfs客户端
* 1.安装nfs客户端
```shell
sudo apt install nfs-common
```
* 2.挂载nfs服务端共享目录
```shell
mount -t nfs 192.168.1.100:/nfs /mnt/nfs # 将192.168.1.100:/nfs挂载到/mnt/nfs目录下
```
* 3.卸载共享目录
```shell
umount /mnt/nfs
```