---
title: "浅谈Python中的with语句"
date: 2017-04-01 18:25:03
tags: [python]
categories: [dev-in-test, python]
---

浅谈Python中的with语句

<!--more-->
** with 语句作为 try/finally 编码范式的一种替代，用于对资源访问进行控制的场合，确保不管使用过程中是否发生异常都会执行必要的“清理”操作，释放资源，比如文件使用后自动关闭、线程中锁的自动获取和释放等。

** 术语
要使用 with 语句，首先要明白上下文管理器这一概念。有了上下文管理器，with 语句才能工作。
下面是一组与上下文管理器和with 语句有关的概念。
上下文管理协议（Context Management Protocol）： 包含方法 __enter__()和 __exit__()，支持该协议的对象要实现这两个方法。
上下文管理器（Context Manager）：支持上下文管理协议的对象，这种对象实现了__enter__()和__exit__() 方法。上下文管理器定义执行 with 语句时要建立的运行时上下文，负责执行 with 语句块上下文中的进入与退出操作。通常使用 with 语句调用上下文管理器，也可以通过直接调用其方法来使用。
运行时上下文（runtime context）：由上下文管理器创建，通过上下文管理器的 __enter__() 和__exit__() 方法实现，__enter__() 方法在语句体执行之前进入运行时上下文，__exit__() 在 语句体执行完后从运行时上下文退出。with 语句支持运行时上下文这一概念。
上下文表达式（Context Expression）：with 语句中跟在关键字 with 之后的表达式，该表达式要返回一个上下文管理器对象。
语句体（with-body）：with 语句包裹起来的代码块，在执行语句体之前会调用上下文管理器的__enter__() 方法，执行完语句体之后会执行 __exit__() 方法。

** 基本语法和工作原理
with 语句的语法格式如下：

清单 1. with 语句的语法格式
with context_expression [as target(s)]:
    with-body
这里 context_expression 要返回一个上下文管理器对象，该对象并不赋值给 as 子句中的 target(s) ，如果指定了 as 子句的话，会将上下文管理器的 __enter__() 方法的返回值赋值给 target(s)。target(s) 可以是单个变量，或者由“()”括起来的元组（不能是仅仅由“,”分隔的变量列表，必须加“()”）。
Python 对一些内建对象进行改进，加入了对上下文管理器的支持，可以用于 with 语句中，比如可以自动关闭文件、线程锁的自动获取和释放等。假设要对一个文件进行操作，使用 with 语句可以有如下代码：

清单 2. 使用 with 语句操作文件对象
{% codeblock lang:python %}
with open(r'somefileName') as somefile:
    for line in somefile:
        print line
        # ...more code
{% endcodeblock %}
这里使用了 with 语句，不管在处理文件过程中是否发生异常，都能保证 with 语句执行完毕后已经关闭了打开的文件句柄。如果使用传统的 try/finally 范式，则要使用类似如下代码：

清单 3. try/finally 方式操作文件对象
{% codeblock lang:python %}
somefile = open(r'somefileName')
try:
    for line in somefile:
        print line
        # ...more code
finally:
    somefile.close()
{% endcodeblock %}
比较起来，使用 with 语句可以减少编码量。已经加入对上下文管理协议支持的还有模块 threading、decimal 等。with 语句的执行过程类似如下代码块：

清单 4. with 语句执行过程
{% codeblock lang:python %}
context_manager = context_expression
exit = type(context_manager).__exit__  
value = type(context_manager).__enter__(context_manager)
exc = True   # True 表示正常执行，即便有异常也忽略；False 表示重新抛出异常，需要对异常进行处理
try:
    try:
        target = value  # 如果使用了 as 子句
        with-body     # 执行 with-body
    except:
        # 执行过程中有异常发生
        exc = False
        # 如果 __exit__ 返回 True，则异常被忽略；如果返回 False，则重新抛出异常
        # 由外层代码对异常进行处理
        if not exit(context_manager, *sys.exc_info()):
            raise
finally:
    # 正常退出，或者通过 statement-body 中的 break/continue/return 语句退出
    # 或者忽略异常退出
    if exc:
        exit(context_manager, None, None, None) 
    # 缺省返回 None，None 在布尔上下文中看做是 False
{% endcodeblock %}

执行 contextexpression，生成上下文管理器 contextmanager
调用上下文管理器的__enter__() 方法；如果使用了 as 子句，则将 __enter__() 方法的返回值赋值给 as 子句中的 target(s)执行语句体 with-body,不管是否执行过程中是否发生了异常，执行上下文管理器的 __exit__() 方法，__exit__() 方法负责执行“清理”工作，如释放资源等。如果执行过程中没有出现异常，或者语句体中执行了语句 break/continue/return，则以 None 作为参数调用 __exit__(None, None, None) ；如果执行过程中出现异常，则使用 sys.excinfo 得到的异常信息为参数调用 __exit__(exctype, excvalue, exctraceback)
出现异常时，如果__exit__(type, value, traceback) 返回 False，则会重新抛出异常，让with 之外的语句逻辑来处理异常，这也是通用做法；如果返回 True，则忽略异常，不再对异常进行处理

** 自定义上下文管理器
开发人员可以自定义支持上下文管理协议的类。自定义的上下文管理器要实现上下文管理协议所需要的__enter__() 和__exit__() 两个方法：
*contextmanager.enter_() ：进入上下文管理器的运行时上下文，在语句体执行前调用。with 语句将该方法的返回值赋值给 as 子句中的 target，如果指定了 as 子句的话
*contextmanager.exit(exctype, excvalue, exctraceback) ：退出与上下文管理器相关的运行时上下文，返回一个布尔值表示是否对发生的异常进行处理。参数表示引起退出操作的异常，如果退出时没有发生异常，则3个参数都为None。如果发生异常，返回True 表示不处理异常，否则会在退出该方法后重新抛出异常以由 with 语句之外的代码逻辑进行处理。如果该方法内部产生异常，则会取代由 statement-body 中语句产生的异常。要处理异常时，不要显示重新抛出异常，即不能重新抛出通过参数传递进来的异常，只需要将返回值设置为 False 就可以了。之后，上下文管理代码会检测是否__exit__() 失败来处理异常
下面通过一个简单的示例来演示如何构建自定义的上下文管理器。注意，上下文管理器必须同时提供__enter__() 和__exit__() 方法的定义，缺少任何一个都会导致 AttributeError；with 语句会先检查是否提供了__exit__() 方法，然后检查是否定义了__enter__() 方法。
假设有一个资源 DummyResource，这种资源需要在访问前先分配，使用完后再释放掉；分配操作可以放到__enter__() 方法中，释放操作可以放到__exit__() 方法中。简单起见，这里只通过打印语句来表明当前的操作，并没有实际的资源分配与释放。

清单 5. 自定义支持 with 语句的对象
{% codeblock lang:python %}
class DummyResource:
def __init__(self, tag):
        self.tag = tag
        print 'Resource [%s]' % tag
    def __enter__(self):
        print '[Enter %s]: Allocate resource.' % self.tag
        return self   # 可以返回不同的对象
    def __exit__(self, exc_type, exc_value, exc_tb):
        print '[Exit %s]: Free resource.' % self.tag
        if exc_tb is None:
            print '[Exit %s]: Exited without exception.' % self.tag
        else:
            print '[Exit %s]: Exited with exception raised.' % self.tag
            return False   # 可以省略，缺省的None也是被看做是Fals
{% endcodeblock %}

DummyResource 中的__enter__() 返回的是自身的引用，这个引用可以赋值给 as 子句中的 target 变量；返回值的类型可以根据实际需要设置为不同的类型，不必是上下文管理器对象本身。
__exit__() 方法中对变量 exc_tb 进行检测，如果不为 None，表示发生了异常，返回 False 表示需要由外部代码逻辑对异常进行处理；注意到如果没有发生异常，缺省的返回值为 None，在布尔环境中也是被看做 False，但是由于没有异常发生__exit__() 的三个参数都为 None，上下文管理代码可以检测这种情况，做正常处理。
下面在 with 语句中访问 DummyResource ：

清单 6. 使用自定义的支持 with 语句的对象
{% codeblock lang:python %}
with DummyResource('Normal'):
    print '[with-body] Run without exceptions.'

with DummyResource('With-Exception'):
    print '[with-body] Run with exception.'
    raise Exception
    print '[with-body] Run with exception. Failed to finish statement-body!'
第1个 with 语句的执行结果如下：
清单 7. with 语句1执行结果
Resource [Normal]
[Enter Normal]: Allocate resource.
[with-body] Run without exceptions.
[Exit Normal]: Free resource.
[Exit Normal]: Exited without exception.
可以看到，正常执行时会先执行完语句体 with-body，然后执行__exit__() 方法释放资源。
第2个 with 语句的执行结果如下：
清单 8. with 语句2执行结果
Resource [With-Exception]
[Enter With-Exception]: Allocate resource.
[with-body] Run with exception.
[Exit With-Exception]: Free resource.
[Exit With-Exception]: Exited with exception raised.

Traceback (most recent call last):
  File "G:/demo", line 20, in <module>
   raise Exception
Exception
{% endcodeblock %}
可以看到，with-body 中发生异常时with-body 并没有执行完，但资源会保证被释放掉，同时产生的异常由 with 语句之外的代码逻辑来捕获处理。
可以自定义上下文管理器来对软件系统中的资源进行管理，比如数据库连接、共享资源的访问控制等。Python 在线文档 Writing Context Managers 提供了一个针对数据库连接进行管理的上下文管理器的简单范例。

