---
title: MCloud常用指令
description: 杭州宏杉学习笔记-MCloud常用指令
#cover: /cover/cover2.png
category: 杭州宏杉科技
tags:
- MCloud
- Cli
  #sticky: 999
published: 2025-07-30
---

* 登录（若未做特殊声明，下述的指令均为在登录的基础上执行）：
  ```shell
  login withAccount accountName=admin password=password
  ```
* 集群开启防火墙：
  ```shell
  globalConfig update name=reject.all.ports category=iptables value=true
  ```
* 开启导出镜像端口
  ```shell
  globalConfig update category=iptables name=mcloud.allow.ports value=8080,60
  ```
* 修改numa值（对应计算规格在线修改）
  ```shell
  globalConfig update category=vm name=numa value=[true/false]
  ```
  
* 对接MacroSAN存储时，若ODSP的版本低于V3.0.19需要修改LUN类型：
  * 若MacroSAN存储上存储池为全HDD Pool，需要修改LUN类型为Thick
  * 其他情况修改LUN类型为Thin
  ```shell
  globalConfig update category=macrosan name=san.lun.mode value=[Thick/Thin]
  ```
  
* 以FC的方式对接MacroSAN时，需要配置FC端口的黑名单
  ```shell 
  globalConfig update category=storageDevice name=FC.storage.blacklist value=50:0b:34:20:03:a2:1e:05,50:0b:34:20:03:a2:06:05(物理端口地址)
  ```
  
* 设置物理机保留内存
  * 以平台为单位(MCloud平台内所有物理机计算保留内存容量一致)：
    ```shell
    gloalConfig update category=kvm name=reservedMemory value=16G
    ```
  * 以集群为单位(MCloud存在多个集群，单个集群内所有物理机计算保留内存容量一致，不同集群间物理机计算保留内存容量不一致)：
    ```shell
    resource updateConfig resourceUuid=[集群uuid] category=kvm name=reservedMemory value=16G
    resource queryConfig resourceUuid=[集群uuid] category=kvm name=reservedMemory  // 查询
    ```
  * 以物理机为单位
    ````shell
    resource updateConfig resourceUuid=[物理机uuid] category=kvm name=reservedMemory value=16G
    ````

* 开启全局容错能力(搭建孪星双节点)
  ```shell
  globalConfig update category=ft name=ft.cluster.enable value=true
  ```