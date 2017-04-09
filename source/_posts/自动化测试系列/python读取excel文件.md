---
title: "python读取excel文件"
date: 2017-04-02 21:56:02
tags: [Web-Test, automation]
categories: [automation]
---
自动化测试中，测试数据一般保存在Excel里，过程产生的数据一般保存在SQL里，上次我们讲里如何读SQL 数据库，今天我们来看下，如何使用Python操作Excel。
<!--more-->
Python操作Excel的library也有好多种，我们拿 Openpyxl为例子，它可以支持Excel2010：
>penpyxl is a Python library for reading and writing Excel 2010 xlsx/xlsm/xltx/xltm files

安装：
{% codeblock lang:python %}
pip install openpyxl
{% endcodeblock %}
用法:
{% codeblock lang:python %}
from openpyxl import load_workbook, Workbook

if __name__ == "__main__":

    # Create a workbook named test and it contains two tabs uat & qa
   file_name = r'c:\test.xlsx'
   wb = Workbook()
    wb.create_sheet('uat',1)
    wb.create_sheet('qa',1)
    wb.save(file_name)

    # Read & write
   wb2 = load_workbook(r'c:\test.xlsx')
    # List all sheets
   print wb2.get_sheet_names()
    # Get target sheet
   s = wb2.get_sheet_by_name('qa')
    # Two ways to set value for specific column/row
   s.cell(row=2,column=1).value= 6
   s['A2'] = 5
   # Get value for specific column/row
   print s.cell(row=2,column=1).value
    print s['A2'].value
{% endcodeblock %}
以上创建了一个workbook，然后又读取这个workbook，并在指定的sheet里设置单元格的值并读出， 代码本身已经解释了一切，无需再特殊说明用法。
这只是Python对Excel的简单实用，如果你有更深层的需求，可以参考官方网页：
[官网](https://openpyxl.readthedocs.io/en/default/index.html)  

通过一周两次发布，我们逐步快把测试用常用的方法技术都讲解完毕了，如果大家想要更深层次的了解某个方法，可以直接回复，我会抽取问题最多的来讲解。