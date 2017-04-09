---
title: "Python多线程 -- Join"
date: 2017-04-01 18:25:06
tags: [python, python多线程系列]
categories: [dev-in-test, python]
---
Python多线程 -- Join

<!--more-->

>join([timeout])
Wait until the thread terminates. This blocks the calling thread until the thread whose join() method is called terminates – either normally or through an unhandled exception – or until the optional timeout occurs.

join方法，阻塞进程直到线程执行完毕。如果一个线程或者一个函数在执行过程中要调用另外一个线程，并且待到其完成以后才能接着执行，那么在调用这个线程时可以使用被调用线程的join方法。

下面通过一段代码来说明join是如何工作的：
{% codeblock lang:python %}
class ThreadExample_1(object):
    def func(self,t):
        print '%s--First method of thread -- start' %t
        t.start()
        t.join()        
        print 'First method of thread -- end'

class ThreadExample_2(threading.Thread):
    def __init__(self):
        threading.Thread.__init__(self)    
        def run(self):
            time.sleep(2)        
            print 'second method of thread'
if __name__ == "__main__":
    t2 = ThreadExample_2()
    t1 = threading.Thread(target=ThreadExample_1().func, args=(t2,))
    t1.start()
{% endcodeblock %}
我们创建了2个子线程，一个t2， 一个t1，其中t1继承自ThreadExample_1,args为t2. 那么先来看下输出：
{% codeblock lang:python %}
<ThreadExample_2(Thread-1, initial)>, First method of thread -- start
second method of thread
First method of thread -- end
{% endcodeblock %}
可以看到， ThreadExample_2的一个实例即t2作为参数传入了主线程的func函数，在第一个print语句输出后，由start()调起了run()并成功进入join阻塞状态，接着，第二个print语句卡在哪里直到t2这个子线程执行完毕才打印出来。

为了进一步说明join的作用我们更改代码如下：
{% codeblock lang:python %}
class ThreadExample_1(object):
    def func(self,t):
        print '%s, First method of thread -- start'%t
        t.start()
        t.join(1)        
        print 'First method of thread -- end'

class ThreadExample_2(threading.Thread):
    def __init__(self):
        threading.Thread.__init__(self)    
    def run(self):
        time.sleep(2)        
        print 'second method of thread't2 = ThreadExample_2()

if __name__ == "__main__":
    t2 = ThreadExample_2()
    t1 = threading.Thread(target=ThreadExample_1().func, args=(t2,))
    t1.start()
{% endcodeblock %}
我们给join一个timeout时间1秒钟， 再来看下运行结果：
{% codeblock lang:python %}
<ThreadExample_2(Thread-1, initial)>, First method of thread -- start
First method of thread -- end
second method of thread
{% endcodeblock %}
可以看到由于join设置了timeout，导致子线程t2没有完成就被退出运行状态，所以主线程的第二个print语句先于t2的print打印出来。
