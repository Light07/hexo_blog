---
title: "Python 多进程"
date: 2017-04-01 18:25:28
tags: [python, python多线程系列]
categories: [dev-in-test]
---
Python 多进程

<!--more-->

前面我们讲了multiple thread的用法， 在Python中，由于GIL（Global Interpreter Lock）的存在（ GIL將会防止多个原生线程同时执行Python字节码。换句话说,GIL將序列化您的所有线程，也就是说，多线程等于时分复用，一个时刻只有一个线程运行），多线程就最多只能用满1个CPU核心，无法真正利用多核的优势。

Python提供了非常好用的多进程包multiprocessing，mutilprocess像线程一样管理进程，这个是mutilprocess的核心，他与threading很是相像，对多核CPU的利用率会比threading好的多。

多进程用法如下：
1. Process
{% codeblock lang:python %}
import multiprocessing
import time
def func(msg):
  for i in range(3):
    print msg
    time.sleep(1)
if __name__ == "__main__":
  p = multiprocessing.Process(target=func, args=("hello", ))
  p.start()
  p.join()
  print "Sub-process done."
{% endcodeblock %}
2. 生产者消费者多进程模型Queue
{% codeblock lang:python %}
import multiprocessing
from multiprocessing import Queue
import random

import time

que = Queue()

class ProduceMethod(multiprocessing.Process):

    def __init__(self, name, que):
        multiprocessing.Process.__init__(self)
        self.data = que
        self.name = "Process%s" %name

    def run(self):
        while True:
            if self.data.qsize() < 2:
                self.do_produce(self.name)
                time.sleep(10)
            else:
                print "queue is %s, stop produce"%self.data.qsize()
                time.sleep(1)

    def do_produce(self, name):
        print "Before produce, the queue size is %s"%self.data.qsize()
        product = random.randint(0, 10)
        self.data.put(product)
        print "%s produce product -- %s, after produce, queue is %s now"%(name, product,self.data.qsize())

class ConsumeMethodEven(multiprocessing.Process):
    def __init__(self, name, que):
        multiprocessing.Process.__init__(self)
        self.data = que
        self.name = "Thread %s" %name

    def run(self):
        while True:

            if self.data.qsize()>0:
                self.do_consume(self.name)
                time.sleep(5)
            else:
                print "queue is %s, stop consume"%self.data.qsize()
                time.sleep(5)

    def do_consume(self, name):
        print"before consume,the quque size is %s"%self.data.qsize()
        product = self.data.get()
        if product %2 ==0:
            print"%s consume product -- %s, after consume, queue is %s now"%(name, product, self.data.qsize())

if __name__ == "__main__":

    for i in range(2):
        p1 = ProduceMethod(i, que)
        p1.start()
    c1 = ConsumeMethodEven(1, que)
    c1.start()
{% endcodeblock %}
由代码可以看出，多进程的方法名和函数名跟多线程基本一模一样，这样就方便了我们开发者（当然内部实现由很多不同），这也是Python容易上手的的一个作证吧。
3. Pool
{% codeblock lang:python %}
import multiprocessing
import time
import os
def func(msg):
    print " Sub-process(es) %s said %s."%(os.getpid(), msg)
    time.sleep(1)
if __name__ == "__main__":
  pool = multiprocessing.Pool(processes=4)
  print "parent id is %s"%os.getpid()
  for i in range(10):
    msg = "hello %d" %(i)
    #注意要用apply_async，如果用apply，就变成阻塞版本了。
    pool.apply_async(func, (msg, ))
  pool.close()
  pool.join()
看一下运行结果：
parent id is 8804
 Sub-process(es) 10948 said hello 0.
 Sub-process(es) 428 said hello 1.
 Sub-process(es) 8592 said hello 2.
 Sub-process(es) 16112 said hello 3.
 Sub-process(es) 10948 said hello 4.
 Sub-process(es) 428 said hello 5.
 Sub-process(es) 8592 said hello 6.
 Sub-process(es) 16112 said hello 7.
 Sub-process(es) 8592 said hello 8.
 Sub-process(es) 428 said hello 9.
{% endcodeblock %}
在运行中，你会看到打印出来的记录是4个一批，打印好4个马上打印下一个4个，那是因为我们设置了processes=4是最多并发进程数量。

何时用多进程：
对于计算密集型程序，多进程并发优于多线程并发。计算密集型程序指的程序的运行时间大部分消耗在CPU的运算处理过程，而硬盘和内存的读写消耗的时间很短；相对地，IO密集型程序指的则是程序的运行时间大部分消耗在硬盘和内存的读写上，CPU的运算时间很短。