---
title: "Jmeter 运用JDBC连接数据库"
tags:
  - API-Test
categories:
  - api-test
date: 2017-04-03 23:57:11
---
上次我们讲了如何利用User Parameters来使用外部Json文件，今天我们讲下一个知识点：如何使用Jmeter连接数据库并且验证数据？
<!--more-->

仍以我们之前的例子来说，最后教师给学生打分完毕后，会有一条数据存在数据库中，那么我们就需要去验证这个数据。
1. <font color="#FF0000">下载最新的JDBC Driver:</font>
>下载最新的JDBC Driver（我们以SQL SERVER为例）
https://msdn.microsoft.com/en-us/library/mt683464.aspx
解压并copy你需要的jar 文件到 jmeter/lib 目录 (重启后生效)

2. 选中你需的Test plan， 把jar文件引入：
![](Jmeter 运用JDBC连接数据库\1.jpg)
3. 右键Thread Group->Add->Config Element->JDBC Connection Configuration
![](Jmeter 运用JDBC连接数据库\2.jpg)
4. 配置如下：
![](Jmeter 运用JDBC连接数据库\3.jpg)
Variable Name:  写个名字，这个名字后面JDBC Request要用。
Database URL:  jdbc:sqlserver://localhost:port;DatabaseName=mydb;  (localhost，port，mydb自行替换)
JDBC Driver Class: com.microsoft.sqlserver.jdbc.SQLServerDriver
Username：数据库连接用户名
Password: 数据库连接密码

5. 在ThreadGroup上右键，Add->Sampler->JDBC Request:
![](Jmeter 运用JDBC连接数据库\4.jpg)
6. 配置如下：
![](Jmeter 运用JDBC连接数据库\5.jpg)

** Variable Name Bound to Pool：配置成与JDBC Connection Configuration里一样的值。
SQL Query：
                      QueryType：看自己需求选择（本例用Prepared Select Statement）。
                       下面input text就填写自己需要的SQl语句。其中“？”代表变量，它的值定义在Parameter values里， 如果有多个，用逗号隔开。
** Parameter values：SQL 语句里？对应的变量，如果有多个，名称以逗号隔开。
** Parameter types：INTEGER, DATE, VARCHAR, DOUBLE
** Variable names: 这个参数用来保存Select 语句返回的值。引用方法参考下扩展阅读。
** Result variable name：创建一个对象变量，保存所有返回的结果

6.       配置好后，run一下看结果：
![](Jmeter 运用JDBC连接数据库\6.jpg)
All passed.

扩展知识：
>Variable names: {% raw %}
如果给这个参数设置了值，它会保存sql语句返回的数据和返回数据的总行数。假如，sql语句返回2行，3列，且variables names设置为A,,C，那么如下变量会被设置为：
　　A_#=2 (总行数)
　　A_1=第1列, 第1行
　　A_2=第1列, 第2行 
　　C_#=2 (总行数) 
　　C_1=第3列, 第1行
　　C_2=第3列, 第2行
如果返回结果为0，那么A_#和C_#会被设置为0，其它变量不会设置值。
如果第一次返回6行数据，第二次只返回3行数据，那么第一次那多的3行数据变量会被清除。
可以使用${A_#}、${A_1}...来获取相应的值
{% endraw %}
