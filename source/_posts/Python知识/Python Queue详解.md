---
title: "Python Queue详解"
date: 2017-04-01 18:25:24
tags: [python, python多线程系列]
categories: [dev-in-test, python]
---

上次我们讲了多线程的condition来实现 生产者消费者模式，今天我们介绍下另外一个方法， Queue, 它是线程安全的，天生利于实现多生产者多消费者模型。

<!--more-->

Python的Queue模块中提供了同步的、线程安全的队列类，包括FIFO（先入先出)队列Queue，LIFO（后入先出）队列LifoQueue，和优先级队列PriorityQueue。这些队列都实现了锁原语，能够在多线程中直接使用。可以使用队列来实现线程间的同步。

我们先来看一个多生产者多消费者的例子：
{% codeblock lang:python %}
import threading
import Queue
import random

import time

con = threading.Condition()
que = Queue.Queue()

class ProduceMethod(threading.Thread):

    def __init__(self, name, que):
        threading.Thread.__init__(self)
        self.data = que
        self.name = "Thread%s" %name

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

class ConsumeMethodEven(threading.Thread):
    def __init__(self, name, que):
        threading.Thread.__init__(self)
        self.data = que
        self.name = "Thread %s" %name

    def run(self):
        while True:

            if self.data.qsize()>0:
                self.do_consume(self.name)
                self.data.task_done()
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
运行结果如下：
Before produce, the queue size is 0
Thread0 produce product -- 4, after produce, queue is 1 now
Before produce, the queue size is 1
Thread1 produce product -- 10, after produce, queue is 2 now
before consume,the quque size is 2
Thread 1 consume product -- 4, after consume, queue is 1 now
before consume,the quque size is 1
Thread 1 consume product -- 10, after consume, queue is 0 now
Before produce, the queue size is 0
......
{% endcodeblock %}
可以看到我们完全不需要自己加锁实现线程同步，Queue的get， 和put操作，自带锁，使用的时候，不用对队列加锁操作。

是不是很方便，再来看下Queue的一些用法：
Queue.qsize() 返回队列的大小
Queue.empty() 如果队列为空，返回True,反之False
Queue.full() 如果队列满了，返回True,反之False。Queue.full 与 maxsize 大小对应
Queue.put(item[, block[, timeout]]) ：
写入队列，timeout等待时间，如果可选的参数block为True且timeout为空对象（默认的情况，阻塞调用，无超时）。
如果timeout是个正整数，阻塞调用进程最多timeout秒，如果一直无空空间可用，抛出Full异常（带超时的阻塞调用）。
如果block为False，如果有空闲空间可用将数据放入队列，否则立即抛出Full异常
Queue.put_nowait(item) 相当Queue.put(item, False)
Queue.get([block[, timeout]])从队列中移除并返回一个数据。block跟timeout参数同put方法
Queue.get_nowait() 相当Queue.get(False)
Queue.task_done()：
意味着之前入队的一个任务已经完成。由队列的消费者线程调用。每一个get()调用得到一个任务，接下来的task_done()调用告诉队列该任务已经处理完毕。
如果当前一个join()正在阻塞，它将在队列中的所有任务都处理完时恢复执行（即每一个由put()调用入队的任务都有一个对应的task_done()调用）join()。
Queue.join()：
阻塞调用线程，直到队列中的所有任务被处理掉。只要有数据被加入队列，未完成的任务数就会增加。当消费者线程调用task_done()（意味着有消费者取得任务并完成任务），未完成的任务数就会减少。当未完成的任务数降到0，join()解除阻塞。实际上意味着等到队列为空，再执行别的操作。