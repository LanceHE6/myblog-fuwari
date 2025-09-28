---
title: Python源码文件打包exe文件

description: 软件项目、产品版本号定义规范
category: 陈年往事
tags:
 - Python
published: 2021-12-22
---

# Python源码文件打包exe文件

##  安装pyinstaller

需要用到的是`pyinstaller` 默认安装的Python是没有这个模块的，需要手动安装

```bash
pip install pyinstaller
```

在控制台输入如上指令安装`pyinstaller`

## 打包

随后在控制台中进入你程序源码所在的文件夹

```bash
cd 路径
```

然后执行命令开始打包

```bash
pyinstaller -F -w -i xx.ico xxx.py
```

`-F` 产生单个的可执行文件

`-w` 指定程序运行时不产生命令行窗口

`-i` 更换生成的exe文件图标（如果不想换就可以不加）

xx.ico 需要更换的图标，需与源码在同一文件夹下或填写完整的图片路径（图片如果是非ico格式需要转换成ico格式才能成功）

更多的执行参数自行百度

打包完成后会在源码目录下生成build ， dist文件夹

另外的pycache 和.spec个人觉得无关紧要（应该不会影响程序的运行）

dist文件夹下就是打包好的exe程序，build是程序的依赖文件
