---
title: "Jmeter 正则表达式提取器传参"
tags:
  - API-Test
categories:
  - api-test
date: 2017-04-03 23:56:57
---
上次我们讲了如何利用Jmeter做接口测试，讲的比较浅，今天来补充一点细节，如何在不同的HTTP Request间传递参数，即前一个请求的输出部分是后一个请求的输入时，应该怎么办
<!--more-->

拿上次的教师打分API为例：
{% codeblock lang:python %}
POST: /survey/evc/EVCSurvey/Save
UAT URL: http://**.englishtown.com/***/Survey/Save
Save Survey Result Request 
{
    "SurveyResult" :   {
        "SurveyTitle" : "ClassTechnicalSurvey",
        "SurveyId" :   1000000,
        "MemberId" :   12345678,
        "ClassId" : "987654321",
        "Token" : "dd556a9d36c1984bbab94f37483f484f",
        "TokenTime" : "2016-07-26T08:56:32.8807460Z"
    },
    "Detail" :   [{
            "Key" :   764, 
            "Value" :   1, 
            "Type" :   3, 
            "Order" :   1
        },{...}
   ]
}
Save   Survey Result Response 
{
    "IsSuccess" : true,
    "Description" : "",
    "StatusCode" :   200 //200->Success, 0->None, -300->Invalid Parameter,
  -400->Invalid Token,   -500->Unhandled Exception
}
{% endcodeblock %}

要使用这个API，我们发送的HTTP Request Body需要一些特定的信息，例如memberid， token， tokenTime等，上次的例子里我们把它们当作一个个变量保存在外部文件里，但显然不太合适，因为不同用户登录后用到的值都不相同，我们最后动态的获取他们。
下面来看看怎么做：
假设我们的整个测试共有3个HTTP请求，第一步是用户登录，第二步是发送一个query string请求，获取服务器返回的memberid，token，tokentime， 第三步就是把第二步返回的请求填入上述request body，然后发送请求，最终完成打分。
显然我们的传参行为发生在第二步和第三部， 我们先看参数获取，在第二步的HTTP Request请求下，右键点击Add->Post Processors->Regular  Expression Extractor
![](Jmeter 正则表达式提取器传参\1.png)
添加后可以看到正则表达式抽取器，我们看如何配置：
![](Jmeter 正则表达式提取器传参\2.png)
位置1：名称及注释
位置2：正则表达式提取内容的范围。（关于各字段的详细说明请查阅协议的相关说明）
位置3：正则表达式提取的相关设置
3.1引用名称：其他地方引用提取值的变量名称，如填写的是：m_id，具体的引用方式是${m_id}
3.2正则表达式：提取内容的正则表达式, 此例中我们需要提取3个参数【稍注意一下： () 表示提取，对于你要提前的内容需要用小括号括起来】。需要注意的是，这3个参数的正则表达式需要是一个，不能分割（就是一个正则匹配所有变量，我本来以为会有分隔符来分割）。
3.3模板：用$$引用起来，如果在正则表达式中有多个提取表达式（多个括号括起来的东东），则可以是$1$，$2$等等，表示解析到的第几个值给m_id，正则表达式的提取模式，值从1开始，值0对应的是整个匹配的表达式
3.4缺省值：正则匹配失败时取缺省值

下面我们给出第二步server的返回值的部分关键内容，结合上图3.2正则表达式的说明，来看下如何填写这个正则表达式：
{% codeblock %}
[{"member_id":23803871,
                ...
"token":"9c0ed9431545dfc3dbaf1e5c7c0ae9ee",
"tokenTime":"2016-07-25T14:41:17.2994676Z",
.....}]
{% endcodeblock %}
由上可以看出，3.2正则表达式我们书写的"member_id":([0-9]*).*"token":("[A-Za-z0-9_]*").*"tokenTime":("[0-9A-Za-z-:.]*") 可以全部匹配merberid，token， tokenTime的内容。
小技巧：可以google在线正则先去确保正则表达式书写正确，然后再按Jmeter语法写入。

拿到参数值后，如何传入下一个HTTP Request呢，非常简单，直接用第二步正则表达式里我们定义的引用名称（Reference Name）就可以了。本例用${m_id_g1}, ${m_id_g2}, ${m_id_g3} 来分别表示不同的参数。如图所示。
![](Jmeter 正则表达式提取器传参\3.png)
我们只定义了m_id一个引用变量，那么m_id_g1, m_id_g2是什么鬼？看下Jmeter语法：

>Suppose you want to match the following portion of a web-page:
name="file.name" value="readme.txt" and you want to extract both file.nameand readme.txt. 
A suitable regular expression would be: 
name="([^"]+)" value="([^"]+)" 
This would create 2 groups, which could be used in the JMeter Regular Expression Extractor template as $1$ and $2$.
The JMeter Regex Extractor saves the values of the groups in additional variables.
For example, assume:
Reference Name: MYREF
Regex: name="(.+?)" value="(.+?)"
Template: $1$$2$
Do not enclose the regular expression in / /
The following variables would be set:
MYREF
file.namereadme.txt
MYREF_g0
name="file.name" value="readme.txt"
MYREF_g1
file.name
MYREF_g2
readme.txt
These variables can be referred to later on in the JMeter test plan, as ${MYREF},${MYREF_g1} etc.

明白了吧，对于本例，tempalte用了$0$, 即取所有匹配的表达式，m_id_g1,则表示匹配的表达式里第一个匹配上的内容，m_id_g2就表示第2个，因为我们有正则匹配上3个符合的内容，g1,g2,g3分别表示这3个。
以上就是如何应用jmeter传参，更多内容敬请关注iTesting。
