---
title: GCC编译器基本用法
description: 介绍几种常用的gcc命令编译C语言程序的使用方法
#image: /cover/cover2.png
tagss: [GCC]
#sticky: 999
published: 2025-01-14
---

# GCC编译器基本用法

## 1.将源文件生成可执行文件

```shell
gcc -o outfile infile
# 或者
gcc infile -o outfile
```

其中`infile`是源文件,`outfile`是即将生成的可执行文件.

**`-o` 与 `outfile` 一定是相邻的**

编译多个文件

```shell
gcc -o outfile infile1 infile2 infile3
```

## 2.将源文件生成目标文件(.o)

```shell
gcc -c infile
```

其中`infile`是源文件,生成的目标文件名为源文件名后加后缀`.o`.
生成的目标文件也可用于静态库或共享库的创建

## 3.生成带有调试信息的可执行文件

```shell
gcc -g infile -o outfile
# 或者
gcc -g -o outfile infile
```

**只有带调试信息的可执行文件才能供GDB调试器进行调试**

## 4.生成汇编文件

```shell
gcc -S infile
```
汇编阶段

## 5.生成`.i`文件

```shell
gcc -E infile -o infile.i
```
预处理阶段
