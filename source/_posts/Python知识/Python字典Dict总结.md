---
title: "Python字典Dict总结"
date: 2017-04-01 18:24:49
tags: [python]
categories: [dev-in-test, python]
---

Python字典Dict总结,

<!--more-->

1. 实现对字典排序
python 字典（dict）的特点就是无序的，按照键（key）来提取相应值（value），如果我们需要字典按值排序的话，怎么办呢？
假设有这么一个字典 ：
{% codeblock lang:python %}
dict = {‘apple’:3, “orange”:2, “oil”:3}
{% endcodeblock %}
*按照key排序
{% codeblock lang:python %}
return_dict = sorted(dict.iteritems(), key=lambda d:d[0], reverse=True)
print return_dict
{% endcodeblock %}
*按照value排序
{% codeblock lang:python %}
return_dict = sorted(dict.iteritems(), key=lambda d:d[1], reverse=True)
print return_dict
{% endcodeblock %}
2. 字典实现一键对应多值
在编码中，比如你得到了一个字典，字典里是一个sprint的每个story及它对应的优先级，那么我们要统计优先级为P的story都有哪些，该怎么办呢？这就涉及到如何利用字典实现一键对应多值。
例如原生字典如下:
{% codeblock lang:python %}
dict  = {"ATEAM-0001": "P1", "ATEAM-0002": "P0", "ATEAM-0003": "P1"}
new_dict = {}
for k, v in dict.iteritems():
    new_dict.setdefault(v,[]).append(k)
print new_dict
{% endcodeblock %}
扩展阅读：
Following is the syntax for setdefault() method −

>dict.setdefault(key, default=None)
##### Parameters
key -- This is the key to be searched.
default -- This is the Value to be returned in case key is not found.
##### Return Value
This method returns the key value available in the dictionary and if given key is not available then it will return provided default value.