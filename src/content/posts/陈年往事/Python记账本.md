---
title: Python记账本

description: 一个基于PySimpleGUI图形界面开发库写的记账本
category: 陈年往事
tags:
 - Python
published: 2022-05-14
---

#  Python记账本

一个基于PySimpleGUI图形界面开发库写的记账本
使用时会在程序目录下生成一个data.txt文件用于存放数据
能简单的实现账单项目的录入及删除
内置导出Excel文件功能，可自定义导出路径并以导出时间命名

```python
import json
import datetime
import PySimpleGUI as sg
import pandas as pd
import os
import xlwt
# import openpyxl

#读取数据
def read_data():
    try:
    with open(r"data.txt", "r") as f:
    json_data=f.read()
    datalist=json.loads(json_data)

        return datalist
    except:
        return []#当文本里面没有数据时返回空列表

#写入数据
def write_data(datalist):
    with open(r"data.txt", "w") as f:
    f.write(json.dumps(datalist, ensure_ascii=False))#防止改变中文编码格式

#显示数据
def show_data():
    try:
        data = read_data()
        datalist = []
        for i in data:
            if i["分类"] == "支出":
                data_list = [i["时间"], i["项目"], i["金额"] * -1, i["分类"]]
                datalist.append(data_list)
            else:
                data_list = [i["时间"], i["项目"], i["金额"], i["分类"]]
                datalist.append(data_list)
                return datalist
            except:
                return []

#总收入支出  1为收入0为支出
def amount_sum(cla):
    try:
        data = read_data()
        sum = 0
        if cla == 1:
            for i in data:
                if i["分类"] == "收入":
                    sum += i["金额"]
        if cla == 0:
            for i in data:
                if i["分类"] == "支出":
                    sum += i["金额"]
        return sum
    except:
        return 0#当文本里面没有数据时返回0

#添加数据
def add_data(project, amount, cla):
    datalist=read_data()
    time=datetime.datetime.now().strftime("%Y/%m/%d")
    data={"时间": time, "项目": project, "金额": amount, "分类": cla}
    datalist.append(data)
    write_data(datalist)
    sg.popup_auto_close('项目录入成功')  # 弹窗

#删除数据
def delete_data(index):
    datalist=read_data()
    del datalist[index]
    write_data(datalist)
    sg.popup_auto_close('删除成功')

#导出Excel文件
def export(path):
    with open(r"data.txt", "r") as f:
        data=f.read()
        f.close()
    with open(r"data.json", "w", encoding='utf-8') as f:
        f.write(data)
        f.close()
    json_data=pd.read_json(r"data.json")
    json_data.to_excel(fr'{path}/{datetime.datetime.now().strftime("%Y%m%d")}.xls')
    os.remove(r"data.json")
    sg.popup("导出成功")

if __name__ == '__main__':
    List=show_data()
    #图形界面布局列表
    sg.theme('LightBrown')
    layout=[
        [sg.T("账目列表：")]+[sg.B("导出Excel文件", key="-export-", pad=((390, 0), (0, 0)))],
        [sg.Table(List,
                headings=["时间", "项目", "金额", "分类"],
                key="-show_table-",
                justification="c",
                auto_size_columns=False,
                def_col_width=15,)],
        [sg.T(f"总收入{amount_sum(1)}元  总支出{amount_sum(0)}元  结余:{amount_sum(1)-amount_sum(0)}元", key="-show_data-")],
        [sg.T("账单项目："), sg.In(key="-project-")],
        [sg.T("账单金额："), sg.In(key="-amount-")],
        [sg.T("项目分类：")] + [sg.Radio(i, group_id=1, key=i) for i in ["收入", "支出"]],#单选框
        [sg.B("提交项目", key="-submit-")]+[sg.B("删除项目", key="-remove-", pad=((430, 0), (0, 0)))]
        
        ]

    window=sg.Window("记账本", layout, font='宋体 13')

    while True:
        event, values =window.read()#获取用户事件
        if event=="-submit-":#用户触发提交数据

            #判断用户是否输入完整数据
            if values["-project-"]=='' or values["-amount-"]=='' or (values["收入"]==False and values["支出"]==False):
                sg.popup("请填写完整的数据")
                continue
            #判断用户输入金额是否为数字
            if not str.isdigit(values["-amount-"]):
                sg.popup("请输入正确的金额")
                continue
            # 获取输入框中的值
            project=values["-project-"]
            amount=float(values["-amount-"])

            for k, v in values.items():
                if v == True:
                    cla=k
                    add_data(project, amount, cla)
                    List=show_data()
                    text=f"总收入{amount_sum(1)}元  总支出{amount_sum(0)}元  结余:{amount_sum(1)-amount_sum(0)}元"
                    window["-show_table-"].update(values=List)
                    window["-show_data-"].update(value=text)
                    #清空输入栏
                    window["-project-"].update(value="")
                    window["-amount-"].update(value="")
                    window["收入"].ResetGroup()

        if event=="-remove-":#用户触发删除数据
            #判断用户是否选中数据
            if values["-show_table-"]:
                #确认删除弹窗
                confirm = sg.popup_ok_cancel("确认删除选中的项目")
                if confirm == "OK":
                    delete_data(values["-show_table-"][0])#删除数据
                    #更新表格
                    List = show_data()
                    text = f"总收入{amount_sum(1)}元  总支出{amount_sum(0)}元  结余:{amount_sum(1) - amount_sum(0)}元"
                    window["-show_table-"].update(values=List)
                    window["-show_data-"].update(value=text)
            else:
                sg.popup("请选中一个项目")

        if event=="-export-":#导出Excel文件
            path=sg.popup_get_folder("保存路径:")

            if path =='':#判断路径是否为空
                sg.popup("请选择正确的路径")
                continue
            elif path is None:#防止用户因取消路径选择而关闭程序
                continue
            else:
                export(path)

        if event in (None, 'Exit'):
            break
    window.close()
```
