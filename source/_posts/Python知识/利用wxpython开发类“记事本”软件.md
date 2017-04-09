---
title: "利用wxpython开发类“记事本”软件"
date: 2017-04-01 18:24:54
tags: [python]
categories: [dev-in-test, python]
---

在学习编程的过程中，你总会忍不住想实现一个小的软件，而当你有这个想法时，你会发现，做软件，GUI是绕不过去的坎，那么python里，有什么优秀的GUI类库，可以极大的提高我们的生产率呢？

<!--more-->

>wxPython is a GUI toolkit for the Python programming language. It allows Python programmers to create programs with a robust, highly functional graphical user interface, simply and easily。

今天我们就来详细解析下如何用wxpython实现一个类似与notepad的软件。
0.开发好后的GUI如下：
![](利用wxpython开发类“记事本”软件\0.gif)
看起来是不是还算可以？那么如何实现呢？
1. 安装wxpython。
http://downloads.sourceforge.net/wxpython/wxPython3.0-win64-3.0.2.0-py27.exe 

2. 使用。
2.1 先写个helloworld对话框来理解基本用法。
{% codeblock lang:python %}
1 #!/usr/bin/env python
2 import wx
3
4 # Create a new app, don't redirect stdout/stderr to a window.
5 app = wx.App(False)  
6
7 # A Frame is a top-level window.
8 frame = wx.Frame(None, wx.ID_ANY, "Hello World") 
9
10 # Show the frame.
11 frame.Show(True)     
12
13 app.MainLoop()
{% endcodeblock %}
Note：
>A wx.Frame is a top-level window. The syntax is x.Frame(Parent, Id, Title). Most of the constructors have this shape (a parent object, followed by an Id). In this example, we use None for "no parent" and wx.ID_ANY to have wxWidgets pick an id for us.
2.2 notepad的实现中的关键点来进行讲解。
2.2.1 把wx.Frame替换成我们的text editor
{% codeblock lang:python %}
   1 #!/usr/bin/env python
   2 import wx
   3 class MyFrame(wx.Frame):
   4     """ We simply derive a new class of Frame. """
   5     def __init__(self, parent, title):
   6         wx.Frame.__init__(self, parent, title=title, size=(200,100))
   7         self.control = wx.TextCtrl(self, style=wx.TE_MULTILINE)
   8         self.Show(True)
   9 
  10 app = wx.App(False)
  11 frame = MyFrame(None, 'Small editor')
  12 app.MainLoop()
 {% endcodeblock %}
2.2.2 text editor有了，还需要在上面做菜单及菜单选项。 
{% codeblock lang:python %}
   1 import wx
   2 
   3 class MainWindow(wx.Frame):
   4     def __init__(self, parent, title):
   5         wx.Frame.__init__(self, parent, title=title, size=(200,100))
   6         self.control = wx.TextCtrl(self, style=wx.TE_MULTILINE)
   7         self.CreateStatusBar() # A Statusbar in the bottom of the window
   8 
   9         # Setting up the menu.
  10         filemenu= wx.Menu()
  11 
  12         # wx.ID_ABOUT and wx.ID_EXIT are standard IDs provided by wxWidgets.
  13         filemenu.Append(wx.ID_ABOUT, "&About"," Information about this program")
  14         filemenu.AppendSeparator()
  15         filemenu.Append(wx.ID_EXIT,"E&xit"," Terminate the program")
  16 
  17         # Creating the menubar.
  18         menuBar = wx.MenuBar()
  19         menuBar.Append(filemenu,"&File") # Adding the "filemenu" to the MenuBar
  20         self.SetMenuBar(menuBar)  # Adding the MenuBar to the Frame content.
  21         self.Show(True)
  22 
  23 app = wx.App(False)
  24 frame = MainWindow(None, "Sample editor")
  25 app.MainLoop()
{% endcodeblock %}
2.2.3 菜单选项有了，但只是GUI不能交互，得加上交互部分。
{% codeblock lang:python %}
   1 class MainWindow(wx.Frame):
   2     def __init__(self, parent, title):
   3         wx.Frame.__init__(self,parent, title=title, size=(200,100))
   4         ...
   5         menuItem = filemenu.Append(wx.ID_ABOUT, "&About"," Information about this program")
   6         self.Bind(wx.EVT_MENU, self.OnAbout, menuItem)

   1 def OnAbout(self, event):
   2         ...
{% endcodeblock %}
2.2.4. 菜单选项有了，每一项菜单选项都有对应的交互函数了，那么运行下看看。如果看到文章开头的界面和功能，则说明运行成功。

3. 上文只是描述了如何借用wxpython进行GUI开发，具体的源代码及实现请参考github：
[Github](https://github.com/Light07/wxpython_example.git)  