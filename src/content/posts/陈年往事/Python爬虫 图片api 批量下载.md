---
title: Python爬虫 图片api 批量下载

description: 一个通过图片api批量下载图片的爬虫
category: 陈年往事
tags:
 - Python
 - 爬虫
published: 2021-12-22
---

# Python爬虫 图片api 批量下载

只能爬图片api！！，较稳定

```python
import urllib.request
import random
import time
import ssl
ssl._create_default_https_context=ssl._create_unverified_context
```

导包，取消验证ssl证书，否则部分网站无法爬取

```python
URL=input("输入爬取图片的网址:>")
n=input("输入爬取图片的数量:>")
n=int(n)
```

用户输入默认为字符串，强制转化为整型数

定义变量用于下载循环
```python
for i in range (n):
    order =lambda : int (round(time.time()*1000*1000))
    #lambda关键字，声明一个匿名函数，返回的时间数值用于图片命名
    response =urllib.request.urlopen(URL)
    #向目标URL发送请求
    cat_img = response.read()
    #接收返回值（二进制）

    time.sleep(2)
    #等待2s
    print ("Download The Picture Successfully and Named:"+str(order()))
    print ("Completed:",round(i/n*100),"%")

    #打印进度
    i +=1
    with open (str(order())+".jpg","wb" ) as f:
        f.write(cat_img)
    #将接受到的内容读取到内存并用jpg格式写入文件夹
    # wb:以二进制格式打开一个文件只用于写入。如果该文件已存在则将其覆盖。如果该文件不存在，创建新文件。

print ("Download Completed")
```

下载循环

