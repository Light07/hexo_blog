---
title: "Jmeter 利用User Defined Variables实现变量重用"
tags:
  - API-Test
categories:
  - api-test
date: 2017-04-03 23:57:03
---
我们上次讲了如何在不同的HTTP Request之间传递参数，今天来看下，如何优化 HTTP Request本身及如何高效使用变量
<!--more-->
我们仍然以下述为例：
>假设我们的整个测试共有3个HTTP请求，第一步是用户登录，第二步是发送一个query string请求，获取服务器返回的memberid，token，tokentime， 第三步步就是把第二步返回的请求填入上述request body，然后发送请求，最终完成打分。

对于我们的每一个HTTP请求，都需要用到Web Server。我们以前是把它们写在不同的请求中间，如果有变化我们需要一个个修改。其实我们可以把它们定义在一个自定义properties文件里读取，方法如下：

1. 在 jmeter.properties 文件中，取消对 user.properties 设置的注释，并 user.properties的值设置为所创建的文件的名称
在 jmeter.properties 文件中指定一个用户属性文件
{% codeblock %}
# Should JMeter automatically load additional JMeter properties?
# File name to look for (comment to disable)
user.properties=myuser.properties
{% endcodeblock %}
2. 更改myuser.properties文件
{% codeblock %}
#----------------------------------------------------------------
# FVT API Test Environment parameters
#----------------------------------------------------------------
#
# --- Login Credentials
USER =test
PASSWORD=test
#
# --- Application Server
APP_SERVER=*.englishtown.com
APP_PORT=80
{% endcodeblock %}
3. 右键ThreadGroup->Add-> Config Element->User Defined Variables
![](Jmeter 利用User Defined Variables实现变量重用\1.png)

4. 配置Name 和Value如下：
![](Jmeter 利用User Defined Variables实现变量重用\2.png)
控制面板的 Value 列中的每个项目的格式为：
>${__property(VARIABLE_NAME,VARIABLE_NAME)}
例如，来自用户属性文件myuser.properties的 APP_SERVER变量被读取为脚本中的 ${__property(APP_SERVER, APP_SERVER)} 函数。

5. 引用时，在每一个HTTP Request的Server or IP里，填写 ${APP_SERVER}就可以了。
user_defined_variable3.png
扩展知识：
>在 JMETER_HOME/bin 目录下的三个文件中定义 JMeter 的属性和变量。在启动 JMeter 时，它会按以下顺序加载这些文件：
jmeter.properties
一个可选的用户定义的属性文件
system.properties
当 JMeter 正在运行的时候，如果要添加任何新属性或改变现有的属性，则必须先关闭 JMeter，然后重新启动它，使更改生效。
jmeter.properties 文件存储与 JMeter 应用程序本身有关的属性。这个文件中仅 保留了特定于 JMeter 程序的属性或特定于框架的属性。创建一个单独的文件（文件名由您选择），用该文件来存储测试环境特定的属性和变量，它们对于和接受测试的应用程序有关联的所有脚本来说是全局的属性和变量 — 例如，管理员的用户名/密码。
