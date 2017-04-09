---
title: "Python多线程系列 -- Condition"
date: 2017-04-01 18:25:21
tags: [python, python多线程系列]
categories: [dev-in-test, python]
---
Python多线程系列 -- Condition

<!--more-->

经过前几次的学习，我们已经明白了多线程的大致用法，我们也可以用lock， rlock 来控制简单的线程同步，那么稍微复杂点的线程同步，比如我需要等到一定条件触发的情况下才处理数据，该怎么办呢？

下面我们以经典的消费者，生产者模型， 来讲解下threading.condition的用法。
{% codeblock lang:python %}
product = None

#Condition

con = threading.Condition()
#Produce method

class ProduceMethod(threading.Thread):

    def __init__(self):
        threading.Thread.__init__(self)    
    def run(self):
        global product        
        if con.acquire():            
            while True:                
                if product is None:
                    print "Produce....."
                    self.do_produce()
                    print "Now we have %s products" %product                    #Notify consumer, product is generated
                    con.notify()                
                 #Wait notify
                 con.wait()
                 time.sleep(2)    
     def do_produce(self):
        global product
        product = 0
        while (product <10):
            product +=1
            print product
     
class ConsumeMethod(threading.Thread):
    def __init__(self):
        threading.Thread.__init__(self)
    def run(self):
        global product
        if con.acquire():
           while True:
              if product is not None:     
                  print "Consume...."
                  self.do_consume()                    
                  #Notify produceser, no product
                  con.notify()                
               #wait notify
               con.wait()
               time.sleep(2)    
     def do_consume(self):
        global product        
        while product>0:
            print product 
            product -= 1  
        if product == 0:
            product = None

if __name__ == "__main__":
    t1 = ProduceMethod()
    t2 = ConsumeMethod()
    t1.start()
    t2.start()
运行结果可以看到，程序在不停的生产，消费。
Produce.....
1
.........
10
Now we have 10 products
Consume....
10
...
1
Produce.....
（继续循环）
{% endcodeblock %}
Condition对象可以在某些事件触发或者达到特定条件后才处理数据，可以把Condiftion理解为一把高级的琐，它提供了比Lock, RLock更高级的功能，允许我们能够控制复杂的线程同步问题。

threadiong.Condition在内部维护一个琐对象（默认是RLock），可以在创建Condition对象的时候把琐对象作为参数传入。Condition也提供了acquire, release方法，其含义与Lock的acquire, release方法一致，其实它只是简单的调用内部琐对象的对应的方法而已。Condition还提供了如下方法(特别要注意：这些方法只有在占用Lock(acquire)之后才能调用，否则将会报RuntimeError异常。)：

Condition.wait([timeout]):
wait方法释放内部所占用的琐，同时线程被挂起，直至接收到通知被唤醒或超时（如果提供了timeout参数的话）。当线程被唤醒并重新占有琐的时候，程序才会继续执行下去。

Condition.notify():
唤醒一个挂起的线程（如果存在挂起的线程）。注意：notify()方法不会释放所占用的琐。

Condition.notify_all()
Condition.notifyAll()
唤醒所有挂起的线程（如果存在挂起的线程）。注意：这些方法不会释放所占用的琐。