---
title: "python多线程系列 -- 线程创建方式"
date: 2017-04-01 18:24:59
tags: [python, python多线程系列]
categories: [dev-in-test]
---
自动化项目中用到到多线程，温故知新之余，我决定花点时间详细讲下Python的多线程模块threading应用。

<!--more-->

今天先讲如何创建线程：
threading模块是在thread之上进行了封装，也是推荐使用的多线程模块，在某些版本中thread模块可能不存在，要使用dump_threading来代替threading模块。

线程创建的两种方式：
>Thisclass represents an activity that is run in a separate thread of control. Thereare two ways to specify the activity: by passing a callable object to theconstructor, or by overriding the run() method in a subclass. Noother methods (except for the constructor) should be overridden in a subclass.In other words, only override the __init__()and run() methods of this class.

>Once a thread object iscreated, its activity must be started by calling the thread’s start() method. This invokes the run() method in a separatethread of control.

直接用代码来区分吧
{% codeblock lang:python %}
#encoding: UTF-8
import threading
import time
class ThreadExample_1(object):
    def func(self):
        print 'First method of thread'

#方法1：将要执行的方法作为参数传给Thread的构造方法

t1 = threading.Thread(target=ThreadExample_1().func)
t1.start()


#方法2：从Thread继承，并重写run()
class ThreadExample_2(threading.Thread):
    def __init__(self):
        threading.Thread.__init__(self)    
    def run(self):
        print 'second method of thread'

t2 = ThreadExample_2()
t2.start()
{% endcodeblock %}