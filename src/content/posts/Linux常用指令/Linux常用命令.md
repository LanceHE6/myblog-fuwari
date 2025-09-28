---
title: Linux系统常用命令
image: ./cover1.jpg
category: Linux
description: 记录常用Linux系统命令
tags:
 - 命令
published: 2024-02-12
---

# Linux系统常用指令

## Linux文件结构

```
/bin        二进制文件，系统常规命令
/boot       系统启动分区，系统启动时读取的文件
/dev        设备文件
/etc        大多数配置文件
/home       普通用户的主目录
/lib        32位函数库
/lib64      64位库
/media      可移动设备(U盘)挂载点
/mnt        临时文件系统挂载点(光驱)
/opt        第三方软件安装位置
/proc       进程信息及硬件信息
/root       超级用户主目录
/sbin       系统管理命令
/srv        数据
/var        存放经常变化的文件(如日志)
/sys        内核相关信息
/tmp        临时文件
/usr        存放用户应用程序和文件
```

## 电源操作

### 立刻关机

```bash
shutdown -h now
# 或者
poweroff
```

### 定时关机

```bash
# 两分钟后关机
shutdown -h 2
```

### 立刻重启

```bash
shutdown -r now
# 或者
reboot
```

### 定时重启

```bash
# 两分钟后重启
shutdown -r 2
```

## 命令帮助

```bash
command --help 	# 查看command命令的用法

man command 	# 打开command命令的说明
```



## 用户和用户组操作

### 用户切换

```bash
su hycer 		# 切换为hycer用户
exit 			# 退出当前用户

sudo -i 		# 切换为root用户
# 或者
su root
```

### 密码修改

```bash
passwd # root用户后面可跟用户名修改对应用户的密码，普通用户只能修改自己的密码
```

### 用户管理

```bash
useradd hycer   # 新增名为hycer的用户

userdel -r hycer # 删除名为hycer的用户账号以及主目录
```

### 用户组管理

```bash
groupadd hycer   # 新增名为hycer的用户组

groupdel hycer # 删除名为hycer的用户组
```

## 目录操作

### 切换目录

```bash
cd 路径 	# 切换到路径
cd / 		# 切换到根目录
cd ~ 		# 切换到home目录
cd .. 		# 切换到父级目录
cd - 		# 切换到上次访问的目录
```

### 查看目录

```bash
  ls                   # 查看当前目录下的所有目录和文件（不包括隐藏的文件）
  ls -a                # 查看当前目录下的所有目录和文件（包括隐藏的文件）
  ls -l                # 列表查看当前目录下的所有目录和文件（列表查看，显示更多信息），与命令"ll"效果一样
  ls /bin              # 查看指定目录下的所有目录和文件 
```

### 创建目录

```bash
  mkdir tools          # 在当前目录下创建一个名为tools的目录
  mkdir /bin/tools     # 在指定目录下创建一个名为tools的目录
```

### 删除目录或文件

```bash
  rm 文件名              # 删除当前目录下的文件
  rm -f 文件名           # 删除当前目录的的文件（不询问）
  rm -r 文件夹名         # 递归删除当前目录下此名的目录
  rm -rf 文件夹名        # 递归删除当前目录下此名的目录（不询问）
  rm -rf *               # 将当前目录下的所有目录和文件全部删除

  rm -rf /*              # 将根目录下的所有文件全部删除【慎用！相当于格式化系统】
```

### 移动或重命名目录或文件

```bash
  mv 当前目录名 新目录名      # 修改目录名，同样适用与文件操作
  mv /usr/tmp/tool /opt       # 将/usr/tmp目录下的tool目录剪切到 /opt目录下面
  mv -r /usr/tmp/tool /opt    # 递归剪切目录中所有文件和文件夹
```

### 拷贝目录

```bash
  cp /usr/tmp/tool /opt       # 将/usr/tmp目录下的tool目录复制到 /opt目录下面
  cp -r /usr/tmp/tool /opt    # 递归剪复制目录中所有文件和文件夹
```

### 搜索目录

```bash
  find /bin -name 'a*'        # 查找/bin目录下的所有以a开头的文件或者目录
```

### 查看当前目录

```
pwd			# 显示当前位置路径
```

## 文件操作

### 新增文件

```bash
touch  a.txt    # 在当前目录下创建名为a的txt文件（文件不存在），如果文件存在，将文件时间属性修改为当前系统时间
```

### 删除文件

```bash
rm 文件名              # 删除当前目录下的文件
rm -f 文件名           # 删除当前目录的的文件（不询问）
rm -r 文件夹		   # 递归删除文件夹下所有文件
```

### 编辑文件（vim）

```bash
vi 文件名              # 打开需要编辑的文件

--进入后，操作界面有三种模式：命令模式（command mode）、插入模式（Insert mode）和底行模式（last line mode）
命令模式
-刚进入文件就是命令模式，通过方向键控制光标位置，
-使用命令"dd"删除当前整行
-使用命令"/字段"进行查找
-按"i"在光标所在字符前开始插入
-按"a"在光标所在字符后开始插入
-按"o"在光标所在行的下面另起一新行插入
-按"："进入底行模式
插入模式
-此时可以对文件内容进行编辑，左下角会显示 "-- 插入 --""
-按"ESC"进入底行模式
底行模式
-退出编辑：      :q
-强制退出：      :q!
-保存并退出：    :wq
## 操作步骤示例 ##
1.保存文件：按"ESC" -> 输入":" -> 输入"wq",回车     # 保存并退出编辑
2.取消操作：按"ESC" -> 输入":" -> 输入"q!",回车     # 撤销本次修改并退出编辑

## 补充 ##
vim +10 filename.txt                   # 打开文件并跳到第10行
vim -R /etc/passwd                     # 以只读模式打开文件
```

### 查看文件

```bash
cat a.txt          # 查看文件最后一屏内容
less a.txt         # PgUp向上翻页，PgDn向下翻页，"q"退出查看
more a.txt         # 显示百分比，回车查看下一行，空格查看上一页，"q"退出查看
tail -100 a.txt    # 查看文件的后100行，"Ctrl+C"退出查看
```

## 文件权限

### 权限说明

  文件权限简介：'r' 代表可读（4），'w' 代表可写（2），'x' 代表执行权限（1），括号内代表"8421法"

  \##文件权限信息示例：-rwxrw-r--

  -第一位：'-'就代表是文件，'d'代表是文件夹

  -第一组三位：拥有者的权限

  -第二组三位：拥有者所在的组，组员的权限

  -第三组三位：代表的是其他用户的权限

### 更改权限

```bash
chmod +x a.txt    
chmod 777 a.txt     # 1+2+4=7，"7"说明授予所有权限
```

### 更改用户组

```bash
chgrp 组 文件
```

### 更改所有者

```bash
chown 用户 文件
```

## 打包与解压

### 拓展名说明

```
.zip、.rar        # windows系统中压缩文件的扩展名
.tar              # Linux中打包文件的扩展名
.gz               # Linux中压缩文件的扩展名
.tar.gz           # Linux中打包并压缩文件的扩展名
```

### 压缩文件

```bash
tar -zcvf 打包压缩后的文件名 要打包的文件
  # 参数说明：z：调用gzip压缩命令进行压缩; c：打包文件; v：显示运行过程; f：指定文件名;
  # 示例：
tar -zcvf a.tar file1 file2,...      # 多个文件压缩打包
```

### 解压文件

```bash
tar -zxvf a.tar                      # 解包至当前目录
tar -zxvf a.tar -C /usr------        # 指定解压的位置
unzip test.zip             # 解压*.zip文件 
unzip -l test.zip          # 查看*.zip文件的内容 
```

## screen

```bash
screen -ls						# 查看所有screen会话
screen -S screenname			# 创建一个新的screen会话
screen -r screenname			# 回到一个screen会话
screen -S screenname -X quit	# 删除一个screen会话
ctrl + a, d						# 从一个screen会话中退出
```

## 其他

```bash
top
htop							# 显示系统进程及资源占用情况

service ssh status 				# 查看服务状态
service ssh restart				# 重启服务

df -h 							# 显示文件系统的磁盘使用情况

uname -a						# 显示系统信息

fdisk -l						# 列出所有磁盘（包括U盘）的分区情况
mount /dev/sdb1 /mnt			# 将文件系统（U盘）挂载到/mnt路径下
umount /mnt						# 解除挂载

ifconfig						# 查看网络配置
iwconfig						# 查看网络设备(网卡)信息

ps -ef         					# 查看所有正在运行的进程

kill pid       					# 杀死该pid的进程
kill -9 pid    					# 强制杀死该进程   
```
