---
title: 使用treer生成文件夹结构

description: 合理地组织项目文件夹结构对于维护项目的可读性和可扩展性至关重要

tags:
 - node工具
 - 结构树
published: 2023-05-28
---

# 使用treer生成文件夹结构

## 安装treer

要开始使用`treer`，首先需要通过`npm`（Node.js包管理器）全局安装它。在你的终端或命令提示符中运行以下命令：

```bash
npm install treer -g
```

## 使用

在需要生成结构的文件夹内输入如下指令:

```bash
treer -e ./structure.txt -i node_modules
```

它就会将目录结构在保存在项目目录中`structure.txt`中并且忽略`node_modules`文件夹

## 效果演示

```
Go_SimpleWMS\go-server
├─config.yaml 				
├─go.mod
├─go.sum
├─main.go 					
├─utils						
├─route						
|   ├─route.go 				
|   ├─group					
├─logs						
├─static					
├─handler					
├─database					
|    ├─myDb					
|    ├─model				
├─config					
```

