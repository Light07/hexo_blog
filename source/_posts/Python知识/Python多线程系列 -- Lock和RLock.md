---
title: "Python多线程系列 -- Lock和RLock"
date: 2017-04-01 18:25:10
tags: [python, python多线程系列]
categories: [dev-in-test, python]
---
Lock和RLock

<!--more-->

先来说下Lock的作用：
用来确保多线程多共享资源的访问。

下面直接看代码， 如果你不用lock：
{% codeblock lang:python %}
x = 0

class ThreadExample_2(threading.Thread):
    def __init__(self):
        threading.Thread.__init__(self)   
 
    def run(self):
        global x    
        time.sleep(1)
        x +=1
        print '%s get the lock, number is %s' \
    % (threading.currentThread().getName(), x)
    
    if __name__ == "__main__":
    lock = threading.Lock()    
    for i in range(5):
        t = ThreadExample_2()
        t.start()
运行结果如下：
Thread-1 get the lock, number is 1
Thread-5 get the lock, number is 2
Thread-4 get the lock, number is 3
Thread-3 get the lock, number is 4
Thread-2 get the lock, number is 5
再次运行一遍，结果如下：
Thread-1 get the lock, number is 1
Thread-2 get the lock, number is 2
Thread-5 get the lock, number is 3
Thread-4 get the lock, number is 4
Thread-3 get the lock, number is 5
两次运行的结果不一样，你肯定不想你的程序随机输出吧？
{% endcodeblock %}
用下lock看看：
{% codeblock lang:python %}
x = 0

class ThreadExample_2(threading.Thread):
    def __init__(self):
        threading.Thread.__init__(self)    
    def run(self):
        global x        
        if lock.acquire():             
             print '%s get the lock.' % threading.currentThread().getName()
             x +=1
             print x
             time.sleep(3)             
             print '%s release lock...' \
             % threading.currentThread().getName()
             lock.release()        
             
if __name__ == "__main__":
    lock = threading.Lock()    
    for i in range(5):
        t = ThreadExample_2()
        t.start()
无论运行几次，都是一个结果：
Thread-1 get the lock.
1
Thread-1 release lock...
Thread-2 get the lock.
2
Thread-2 release lock...
Thread-3 get the lock.
3
Thread-3 release lock...
Thread-4 get the lock.
4
Thread-4 release lock...
Thread-5 get the lock.
5
Thread-5 release lock...
{% endcodeblock %}
解释：
Lock对象的状态可以为locked和unlocked
使用acquire()设置为locked状态；
使用release()设置为unlocked状态。
如果当前的状态为unlocked，则acquire()会将状态改为locked然后立即返回。当状态为locked的时候，acquire()将被阻塞直到另一个线程中调用release()来将状态改为unlocked，然后acquire()才可以再次将状态置为locked。

Lock.acquire(blocking=True, timeout=-1),blocking参数表示是否阻塞当前线程等待，timeout表示阻塞时的等待时间 。如果成功地获得lock，则acquire()函数返回True，否则返回False，timeout超时时如果还没有获得lock仍然返回False。

那么Rlock呢？
RLock与Lock的区别是：RLock中除了状态locked和unlocked外还记录了当前lock的owner和递归层数，使得RLock可以被同一个线程多次acquire()。

还是来看代码：
{% codeblock lang:python %}
lock = threading.Lock() 
#Lock对象
lock.acquire()
lock.acquire()  
#产生了死琐
lock.release()
lock.release()
运行下就看到程序死锁了，永远不会结束。
再来看Rlock：
lock = threading.RLock() 

#可以重复lock

lock.acquire()
lock.acquire()  

lock.release()
lock.release()
print "done without problem"
{% endcodeblock %}
输出结果“done without problem” 正常。
