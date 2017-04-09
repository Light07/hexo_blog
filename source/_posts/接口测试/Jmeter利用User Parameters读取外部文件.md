---
title: "Jmeter利用User Parameters读取外部文件"
tags:
  - API-Test
categories:
  - api-test
date: 2017-04-03 23:57:07
---
上次我们讲了如何最优化的重用变量，我们使用了User Defined Variables，这些变量一般都是全局的，或者是常量， 假设我们用到的变量来自上一个请求，那么我们如何使用呢？（还记得前面的分享< Jmeter 正则表达式提取器传参>吗？）另外，我们发送的HTTP Request bodydata里，经常需要发送一大串的Json数据,我们之前是写到每一个请求的body里，那么有没有办法写到外部文件里以重用呢？
<!--more-->
今天我们来讲如何使用User Parameters读取外部文件。
仍以我们之前的例子来说：
>假设我们的整个测试共有3个HTTP请求，第一步是用户登录，第二步是发送一个query string请求，获取服务器返回的memberid，token，tokentime， 第三步步就是把第二步返回的请求填入上述request body，然后发送请求，最终完成打分。

第二步和第三步之间的参数传递我们已经在文章< Jmeter 正则表达式提取器传参>里实现了，第三部分的request body Data我们之前是放到一个个具体的HTTPRequest Sampler里的，这种方法的一个常见问题是，如果 JSON 结构（或数据）发现变化（也许是因为 API 参数的更改），那么您必须进入 JMeter 测试，找到 HTTP 请求的主体，并修改 JSON 结构（和数据），以满足新的要求。

那么如何解决这个问题呢？

我们以前的body data如下：
{% codeblock %}
{
    "SurveyResult" : {
        "SurveyTitle" : "ClassTechnicalSurvey",
        "SurveyId" : 104,
        "MemberId" : "${m_id_g1}",
     "ClassId" : "1422994",
        "Token" :  ${m_id_g2},
        "TokenTime" : ${m_id_g3}
    },
"Detail" : [{…}]
}
{% endcodeblock %}
1. 我们新建立一个json文件，名称customizejson.json，保存在目录..\ _testcase\jsontemplates下，
内容如下：
{% codeblock %}
{
    "SurveyResult" : {
        "SurveyTitle" : "ClassTechnicalSurvey",
        "SurveyId" : 104,
        "MemberId" : "${__eval(${m_id_g1})}",
        "ClassId" : "1422994",
        "Token" : ${__eval(${m_id_g2})},
        "TokenTime" : ${__eval(${m_id_g3})}
    },
"Detail" : [{…}]
}
{% endcodeblock %}
上述文件中的 JSON 为每个已定义的替代变量调用了 JMeter __eval() 函数。增加的这一步骤使得在运行时执行 JMeter 脚本的时候可以对变量进行计算。

2. 在HTTP Request Smapler里，设置如下：
![](Jmeter利用User Parameters读取外部文件\1.png)
可以看到以前我们写在BodyData里的JSON数据被 ${__eval(${CUSTOMIZE_JSON})代替。

3. 在这个Sampler上，右键点击Add->Pre processors->User Parameters
![](Jmeter利用User Parameters读取外部文件\2.png)

4. 配置User Parameters 如下：
![](Jmeter利用User Parameters读取外部文件\3.png)
以上定义的CUSTOMIZE_JSON 变量就是前面步骤里在BodyData里填写的变量，它被设置为 FileToString()，并使用指向 JSON 模板文件的路径作为参数。JSON 实体模板文件的所有内容都被读入 CUSTOMIZE_JSON变量。因此，JSON 实体模板文件的内容是在运行时计算的，所定义的所有替代字符串都被翻译成为它们定义的数据。

另外两个变量定义在myuser.properties文件里：
>#--Json template（路径不需要用引号括起来）
JSONTEMPLATE_ROOT=C:\\apache-jmeter-2.13\\_testcase\\jsontemplates
CSVDATA_ROOT=C:\\apache-jmeter-2.13\\_testcase\\jmeter_config.txt

按上述步骤设置好，run一下，我们看看结果：
![](Jmeter利用User Parameters读取外部文件\4.png)
全部pass，并且相应的动态变量也被正确获取替换，成功！

扩展知识：
>__FileToString¶
The FileToString function can be used to read an entire file. Each time it is called it reads the entire file.
If an error occurs opening or reading the file, then the function returns the string "**ERR**"

>__eval
The eval function returns the result of evaluating a string expression.
This allows one to interpolate variable and function references in a string which is stored in a variable. For example, given the following variables:
name=Smith
column=age
table=birthdays
SQL=select ${column} from ${table} where name='${name}'
then ${__eval(${SQL})} will evaluate as "select age from birthdays where name='Smith'".
This can be used in conjunction with CSV Dataset, for example where the both SQL statements and the values are defined in the data file. 
